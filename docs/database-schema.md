# CacaoSense Database Schema

## Overview

CacaoSense uses PostgreSQL (via Supabase) as its primary database. The schema is designed to support supply chain tracking, forecasting, and business intelligence for cacao processors.

---

## Entity Relationship Diagram

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Processors │────<│   Farmers   │────<│   Harvests  │
└─────────────┘     └─────────────┘     └─────────────┘
       │                   │                   │
       │                   │                   │
       ▼                   ▼                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Products   │     │   Scores    │     │ Predictions │
└─────────────┘     └─────────────┘     └─────────────┘
       │
       │
       ▼
┌─────────────┐     ┌─────────────┐
│   Orders    │     │  Inventory  │
└─────────────┘     └─────────────┘
```

---

## Core Tables

### processors

Cacao processing businesses (primary users of the dashboard).

```sql
CREATE TABLE processors (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    
    -- Basic Info
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    
    -- Business Details
    business_name VARCHAR(255),
    business_type VARCHAR(50), -- 'sole_proprietor', 'partnership', 'corporation'
    address TEXT,
    city VARCHAR(100),
    province VARCHAR(100),
    
    -- Settings
    settings JSONB DEFAULT '{}',
    subscription_tier VARCHAR(50) DEFAULT 'starter', -- 'starter', 'professional', 'enterprise'
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Supabase Auth
    auth_user_id UUID REFERENCES auth.users(id)
);

-- Indexes
CREATE INDEX idx_processors_email ON processors(email);
CREATE INDEX idx_processors_auth_user ON processors(auth_user_id);
```

---

### farmers

Smallholder cacao farmers who supply beans.

```sql
CREATE TABLE farmers (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    processor_id UUID NOT NULL REFERENCES processors(id) ON DELETE CASCADE,
    
    -- Basic Info
    name VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    messenger_id VARCHAR(100), -- Facebook Messenger PSID
    
    -- Location
    barangay VARCHAR(100),
    municipality VARCHAR(100),
    province VARCHAR(100) DEFAULT 'Davao',
    coordinates POINT, -- GPS coordinates
    
    -- Farm Details
    farm_size_hectares DECIMAL(10,2),
    tree_count INTEGER,
    tree_age_years INTEGER,
    cacao_variety VARCHAR(100), -- 'Trinitario', 'Criollo', 'Forastero'
    
    -- Status
    status VARCHAR(50) DEFAULT 'active', -- 'active', 'inactive', 'pending'
    registration_date DATE DEFAULT CURRENT_DATE,
    
    -- Scores (computed)
    reliability_score DECIMAL(5,2) DEFAULT 50.00,
    quality_score DECIMAL(5,2) DEFAULT 50.00,
    engagement_score DECIMAL(5,2) DEFAULT 50.00,
    overall_score DECIMAL(5,2) GENERATED ALWAYS AS (
        (reliability_score * 0.4 + quality_score * 0.4 + engagement_score * 0.2)
    ) STORED,
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_report_at TIMESTAMP WITH TIME ZONE,
    last_delivery_at TIMESTAMP WITH TIME ZONE
);

-- Indexes
CREATE INDEX idx_farmers_processor ON farmers(processor_id);
CREATE INDEX idx_farmers_messenger ON farmers(messenger_id);
CREATE INDEX idx_farmers_status ON farmers(status);
CREATE INDEX idx_farmers_scores ON farmers(overall_score DESC);
```

---

### harvest_reports

Farmer-submitted harvest reports via Messenger or SMS.

```sql
CREATE TABLE harvest_reports (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    farmer_id UUID NOT NULL REFERENCES farmers(id) ON DELETE CASCADE,
    processor_id UUID NOT NULL REFERENCES processors(id) ON DELETE CASCADE,
    
    -- Report Details
    reported_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    source VARCHAR(50) DEFAULT 'messenger', -- 'messenger', 'sms', 'manual'
    raw_message TEXT, -- Original message from farmer
    
    -- Harvest Information
    estimated_volume_kg DECIMAL(10,2) NOT NULL,
    readiness_days INTEGER DEFAULT 0, -- Days until ready for pickup
    estimated_pickup_date DATE,
    
    -- Quality
    quality_rating VARCHAR(20), -- 'excellent', 'good', 'fair', 'poor'
    fermentation_days INTEGER,
    moisture_content DECIMAL(5,2), -- Percentage
    
    -- Issues
    has_issues BOOLEAN DEFAULT FALSE,
    issue_description TEXT,
    
    -- Actual (filled after delivery)
    actual_volume_kg DECIMAL(10,2),
    actual_quality VARCHAR(20),
    delivered_at TIMESTAMP WITH TIME ZONE,
    
    -- Status
    status VARCHAR(50) DEFAULT 'pending', -- 'pending', 'confirmed', 'delivered', 'cancelled'
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_harvest_reports_farmer ON harvest_reports(farmer_id);
CREATE INDEX idx_harvest_reports_processor ON harvest_reports(processor_id);
CREATE INDEX idx_harvest_reports_date ON harvest_reports(estimated_pickup_date);
CREATE INDEX idx_harvest_reports_status ON harvest_reports(status);
```

---

### products

Cacao products that the processor manufactures.

```sql
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    processor_id UUID NOT NULL REFERENCES processors(id) ON DELETE CASCADE,
    
    -- Product Info
    name VARCHAR(100) NOT NULL,
    sku VARCHAR(50),
    description TEXT,
    category VARCHAR(50), -- 'tablea', 'chocolate_bar', 'powder', 'nibs', 'butter'
    
    -- Production
    beans_per_unit_kg DECIMAL(10,3) NOT NULL, -- kg of beans needed per unit
    production_time_hours DECIMAL(10,2), -- Time to produce one unit
    shelf_life_days INTEGER,
    
    -- Pricing
    unit_price DECIMAL(10,2) NOT NULL,
    production_cost DECIMAL(10,2) NOT NULL,
    margin DECIMAL(10,2) GENERATED ALWAYS AS (unit_price - production_cost) STORED,
    margin_percentage DECIMAL(5,2) GENERATED ALWAYS AS (
        CASE WHEN unit_price > 0 THEN ((unit_price - production_cost) / unit_price * 100) ELSE 0 END
    ) STORED,
    
    -- Status
    is_active BOOLEAN DEFAULT TRUE,
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_products_processor ON products(processor_id);
CREATE INDEX idx_products_category ON products(category);
CREATE INDEX idx_products_active ON products(is_active);
```

---

### orders

Customer orders for products.

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    processor_id UUID NOT NULL REFERENCES processors(id) ON DELETE CASCADE,
    
    -- Order Info
    order_number VARCHAR(50) UNIQUE,
    customer_name VARCHAR(255) NOT NULL,
    customer_contact VARCHAR(100),
    customer_email VARCHAR(255),
    
    -- Delivery
    delivery_address TEXT,
    delivery_date DATE,
    
    -- Status
    status VARCHAR(50) DEFAULT 'pending', -- 'pending', 'confirmed', 'in_production', 'ready', 'delivered', 'cancelled'
    priority VARCHAR(20) DEFAULT 'normal', -- 'low', 'normal', 'high', 'urgent'
    
    -- Totals
    subtotal DECIMAL(12,2) DEFAULT 0,
    discount DECIMAL(12,2) DEFAULT 0,
    total DECIMAL(12,2) DEFAULT 0,
    
    -- Notes
    notes TEXT,
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    confirmed_at TIMESTAMP WITH TIME ZONE,
    delivered_at TIMESTAMP WITH TIME ZONE
);

-- Indexes
CREATE INDEX idx_orders_processor ON orders(processor_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_delivery_date ON orders(delivery_date);
CREATE INDEX idx_orders_number ON orders(order_number);
```

---

### order_items

Line items for each order.

```sql
CREATE TABLE order_items (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id UUID NOT NULL REFERENCES products(id),
    
    -- Item Details
    quantity INTEGER NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    total_price DECIMAL(12,2) GENERATED ALWAYS AS (quantity * unit_price) STORED,
    
    -- Production
    beans_required_kg DECIMAL(10,2), -- Calculated from product.beans_per_unit_kg * quantity
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_order_items_product ON order_items(product_id);
```

---

### supply_predictions

AI-generated supply forecasts.

```sql
CREATE TABLE supply_predictions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    processor_id UUID NOT NULL REFERENCES processors(id) ON DELETE CASCADE,
    
    -- Prediction Details
    prediction_date DATE NOT NULL, -- Date when prediction was made
    target_date DATE NOT NULL, -- Date being predicted
    
    -- Forecast
    predicted_volume_kg DECIMAL(10,2) NOT NULL,
    confidence_level DECIMAL(3,2), -- 0.00 to 1.00
    confidence_category VARCHAR(20), -- 'high', 'medium', 'low'
    
    -- Breakdown
    breakdown_by_farmer JSONB, -- {farmer_id: volume}
    
    -- Model Info
    model_version VARCHAR(50),
    features_used JSONB,
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_predictions_processor ON supply_predictions(processor_id);
CREATE INDEX idx_predictions_target ON supply_predictions(target_date);
CREATE UNIQUE INDEX idx_predictions_unique ON supply_predictions(processor_id, prediction_date, target_date);
```

---

### inventory

Current inventory levels.

```sql
CREATE TABLE inventory (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    processor_id UUID NOT NULL REFERENCES processors(id) ON DELETE CASCADE,
    
    -- Item Type
    item_type VARCHAR(50) NOT NULL, -- 'raw_beans', 'fermented_beans', 'roasted_beans', 'finished_product'
    product_id UUID REFERENCES products(id), -- For finished products
    
    -- Quantity
    quantity DECIMAL(12,2) NOT NULL,
    unit VARCHAR(20) DEFAULT 'kg',
    
    -- Location
    location VARCHAR(100), -- 'warehouse_a', 'processing', etc.
    
    -- Quality
    batch_number VARCHAR(50),
    received_date DATE,
    expiry_date DATE,
    
    -- Timestamps
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_inventory_processor ON inventory(processor_id);
CREATE INDEX idx_inventory_type ON inventory(item_type);
CREATE INDEX idx_inventory_product ON inventory(product_id);
```

---

### price_history

Historical price tracking for products and raw materials.

```sql
CREATE TABLE price_history (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    processor_id UUID NOT NULL REFERENCES processors(id) ON DELETE CASCADE,
    
    -- Price Type
    price_type VARCHAR(50) NOT NULL, -- 'raw_beans', 'product_sale', 'market_rate'
    product_id UUID REFERENCES products(id),
    
    -- Price
    price DECIMAL(10,2) NOT NULL,
    unit VARCHAR(20) DEFAULT 'kg',
    
    -- Source
    source VARCHAR(100), -- 'manual', 'market', 'supplier_quote'
    notes TEXT,
    
    -- Timestamp
    recorded_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_price_history_processor ON price_history(processor_id);
CREATE INDEX idx_price_history_type ON price_history(price_type);
CREATE INDEX idx_price_history_date ON price_history(recorded_at);
```

---

### farmer_scores_history

Historical tracking of farmer scores.

```sql
CREATE TABLE farmer_scores_history (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    farmer_id UUID NOT NULL REFERENCES farmers(id) ON DELETE CASCADE,
    
    -- Scores
    reliability_score DECIMAL(5,2),
    quality_score DECIMAL(5,2),
    engagement_score DECIMAL(5,2),
    overall_score DECIMAL(5,2),
    
    -- Context
    calculation_details JSONB, -- Details of how scores were calculated
    
    -- Timestamp
    recorded_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_score_history_farmer ON farmer_scores_history(farmer_id);
CREATE INDEX idx_score_history_date ON farmer_scores_history(recorded_at);
```

---

### alerts

System-generated alerts and notifications.

```sql
CREATE TABLE alerts (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    processor_id UUID NOT NULL REFERENCES processors(id) ON DELETE CASCADE,
    
    -- Alert Info
    alert_type VARCHAR(50) NOT NULL, -- 'supply_dip', 'farmer_inactive', 'price_change', 'low_inventory', 'order_due'
    severity VARCHAR(20) DEFAULT 'info', -- 'info', 'warning', 'critical'
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    
    -- Related Entities
    related_farmer_id UUID REFERENCES farmers(id),
    related_order_id UUID REFERENCES orders(id),
    related_product_id UUID REFERENCES products(id),
    
    -- Status
    is_read BOOLEAN DEFAULT FALSE,
    is_dismissed BOOLEAN DEFAULT FALSE,
    
    -- Action
    action_url VARCHAR(255),
    action_label VARCHAR(100),
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    read_at TIMESTAMP WITH TIME ZONE,
    expires_at TIMESTAMP WITH TIME ZONE
);

-- Indexes
CREATE INDEX idx_alerts_processor ON alerts(processor_id);
CREATE INDEX idx_alerts_type ON alerts(alert_type);
CREATE INDEX idx_alerts_unread ON alerts(processor_id, is_read) WHERE NOT is_read;
```

---

### weather_data

Cached weather data for forecasting.

```sql
CREATE TABLE weather_data (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    
    -- Location
    location VARCHAR(100) NOT NULL,
    latitude DECIMAL(10,6),
    longitude DECIMAL(10,6),
    
    -- Weather
    date DATE NOT NULL,
    temperature_min DECIMAL(5,2),
    temperature_max DECIMAL(5,2),
    temperature_avg DECIMAL(5,2),
    humidity DECIMAL(5,2),
    precipitation_mm DECIMAL(10,2),
    weather_code INTEGER,
    
    -- Source
    source VARCHAR(50) DEFAULT 'open-meteo',
    
    -- Timestamp
    fetched_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_weather_location ON weather_data(location);
CREATE INDEX idx_weather_date ON weather_data(date);
CREATE UNIQUE INDEX idx_weather_unique ON weather_data(location, date);
```

---

### conversation_logs

Messenger conversation history with farmers.

```sql
CREATE TABLE conversation_logs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    farmer_id UUID REFERENCES farmers(id) ON DELETE SET NULL,
    
    -- Message
    messenger_id VARCHAR(100) NOT NULL,
    direction VARCHAR(10) NOT NULL, -- 'inbound', 'outbound'
    message_type VARCHAR(50), -- 'text', 'image', 'quick_reply'
    content TEXT,
    
    -- NLP
    intent VARCHAR(100),
    entities JSONB,
    confidence DECIMAL(3,2),
    
    -- Timestamp
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_conversation_farmer ON conversation_logs(farmer_id);
CREATE INDEX idx_conversation_messenger ON conversation_logs(messenger_id);
CREATE INDEX idx_conversation_date ON conversation_logs(created_at);
```

---

## Views

### v_supply_overview

Aggregated supply view for dashboard.

```sql
CREATE VIEW v_supply_overview AS
SELECT 
    processor_id,
    DATE(estimated_pickup_date) as pickup_date,
    COUNT(*) as report_count,
    SUM(estimated_volume_kg) as total_estimated_kg,
    SUM(actual_volume_kg) as total_actual_kg,
    AVG(CASE 
        WHEN actual_volume_kg IS NOT NULL AND estimated_volume_kg > 0 
        THEN actual_volume_kg / estimated_volume_kg 
        ELSE NULL 
    END) as accuracy_ratio
FROM harvest_reports
WHERE status != 'cancelled'
GROUP BY processor_id, DATE(estimated_pickup_date);
```

### v_farmer_performance

Farmer performance metrics.

```sql
CREATE VIEW v_farmer_performance AS
SELECT 
    f.id as farmer_id,
    f.name,
    f.processor_id,
    f.overall_score,
    COUNT(hr.id) as total_reports,
    SUM(hr.actual_volume_kg) as total_delivered_kg,
    AVG(hr.actual_volume_kg) as avg_delivery_kg,
    COUNT(CASE WHEN hr.actual_volume_kg >= hr.estimated_volume_kg * 0.9 THEN 1 END)::FLOAT / 
        NULLIF(COUNT(hr.actual_volume_kg), 0) * 100 as delivery_accuracy_pct,
    MAX(hr.delivered_at) as last_delivery
FROM farmers f
LEFT JOIN harvest_reports hr ON f.id = hr.farmer_id AND hr.status = 'delivered'
GROUP BY f.id, f.name, f.processor_id, f.overall_score;
```

---

## Functions

### update_farmer_scores()

Recalculate farmer scores based on recent performance.

```sql
CREATE OR REPLACE FUNCTION update_farmer_scores(p_farmer_id UUID)
RETURNS VOID AS $$
DECLARE
    v_reliability DECIMAL(5,2);
    v_quality DECIMAL(5,2);
    v_engagement DECIMAL(5,2);
BEGIN
    -- Calculate reliability (based on delivery accuracy)
    SELECT COALESCE(
        AVG(CASE 
            WHEN actual_volume_kg >= estimated_volume_kg * 0.9 THEN 100
            WHEN actual_volume_kg >= estimated_volume_kg * 0.7 THEN 70
            WHEN actual_volume_kg >= estimated_volume_kg * 0.5 THEN 50
            ELSE 30
        END), 50
    ) INTO v_reliability
    FROM harvest_reports
    WHERE farmer_id = p_farmer_id 
    AND status = 'delivered'
    AND delivered_at > NOW() - INTERVAL '90 days';
    
    -- Calculate quality (based on quality ratings)
    SELECT COALESCE(
        AVG(CASE actual_quality
            WHEN 'excellent' THEN 100
            WHEN 'good' THEN 80
            WHEN 'fair' THEN 60
            WHEN 'poor' THEN 30
            ELSE 50
        END), 50
    ) INTO v_quality
    FROM harvest_reports
    WHERE farmer_id = p_farmer_id 
    AND status = 'delivered'
    AND delivered_at > NOW() - INTERVAL '90 days';
    
    -- Calculate engagement (based on reporting frequency)
    SELECT COALESCE(
        CASE 
            WHEN COUNT(*) >= 12 THEN 100
            WHEN COUNT(*) >= 8 THEN 80
            WHEN COUNT(*) >= 4 THEN 60
            WHEN COUNT(*) >= 1 THEN 40
            ELSE 20
        END, 20
    ) INTO v_engagement
    FROM harvest_reports
    WHERE farmer_id = p_farmer_id
    AND reported_at > NOW() - INTERVAL '90 days';
    
    -- Update farmer scores
    UPDATE farmers
    SET 
        reliability_score = v_reliability,
        quality_score = v_quality,
        engagement_score = v_engagement,
        updated_at = NOW()
    WHERE id = p_farmer_id;
    
    -- Log score history
    INSERT INTO farmer_scores_history (
        farmer_id, reliability_score, quality_score, engagement_score, overall_score
    )
    SELECT id, reliability_score, quality_score, engagement_score, overall_score
    FROM farmers WHERE id = p_farmer_id;
    
END;
$$ LANGUAGE plpgsql;
```

---

## Row Level Security (RLS)

Enable RLS for multi-tenant security:

```sql
-- Enable RLS on all tables
ALTER TABLE farmers ENABLE ROW LEVEL SECURITY;
ALTER TABLE harvest_reports ENABLE ROW LEVEL SECURITY;
ALTER TABLE products ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE inventory ENABLE ROW LEVEL SECURITY;
ALTER TABLE alerts ENABLE ROW LEVEL SECURITY;

-- Processor can only see their own data
CREATE POLICY processor_isolation ON farmers
    FOR ALL USING (processor_id = auth.uid());

CREATE POLICY processor_isolation ON harvest_reports
    FOR ALL USING (processor_id = auth.uid());

CREATE POLICY processor_isolation ON products
    FOR ALL USING (processor_id = auth.uid());

CREATE POLICY processor_isolation ON orders
    FOR ALL USING (processor_id = auth.uid());

CREATE POLICY processor_isolation ON inventory
    FOR ALL USING (processor_id = auth.uid());

CREATE POLICY processor_isolation ON alerts
    FOR ALL USING (processor_id = auth.uid());
```

---

## Triggers

### Auto-update timestamps

```sql
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_farmers_timestamp
    BEFORE UPDATE ON farmers
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_products_timestamp
    BEFORE UPDATE ON products
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_orders_timestamp
    BEFORE UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

### Generate order number

```sql
CREATE OR REPLACE FUNCTION generate_order_number()
RETURNS TRIGGER AS $$
BEGIN
    NEW.order_number = 'ORD-' || TO_CHAR(NOW(), 'YYYYMMDD') || '-' || 
        LPAD(nextval('order_number_seq')::TEXT, 4, '0');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE SEQUENCE order_number_seq;

CREATE TRIGGER set_order_number
    BEFORE INSERT ON orders
    FOR EACH ROW EXECUTE FUNCTION generate_order_number();
```

---

## Sample Data

See `docs/sample-data.sql` for sample data to populate the database for development and testing.

---

## Migration Notes

- Always use migrations for schema changes
- Test migrations on staging before production
- Keep backwards compatibility when possible
- Document breaking changes