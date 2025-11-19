# ES|QL (Elasticsearch Query Language) Cheat Sheet

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Query Structure](#basic-query-structure)
3. [Source Commands](#source-commands)
4. [Processing Commands](#processing-commands)
5. [Data Selection & Filtering](#data-selection--filtering)
6. [Aggregations & Statistics](#aggregations--statistics)
7. [String Functions](#string-functions)
8. [Mathematical Functions](#mathematical-functions)
9. [Date/Time Functions](#datetime-functions)
10. [Type Conversion Functions](#type-conversion-functions)
11. [Conditional Functions](#conditional-functions)
12. [Array Functions](#array-functions)
13. [Join Operations (LOOKUP)](#join-operations-lookup)
14. [Sorting & Limiting](#sorting--limiting)
15. [Grouping & Aggregating](#grouping--aggregating)
16. [Advanced Patterns](#advanced-patterns)

---

## Introduction

ES|QL is Elasticsearch's new query language designed for data exploration, analysis, and transformation. It uses a pipe (`|`) syntax to chain commands together.

**Basic Syntax:**
```
FROM <data-source>
| <processing-command>
| <processing-command>
| ...
```

---

## Basic Query Structure

### Simple Query
```esql
FROM logs-*
| LIMIT 10
```

### Query with Multiple Commands
```esql
FROM employees
| WHERE department == "Engineering"
| KEEP name, salary, hire_date
| SORT salary DESC
| LIMIT 5
```

---

## Source Commands

### FROM - Specify Data Source
```esql
// From a single index
FROM logs-2024

// From multiple indices with wildcard
FROM logs-*, metrics-*

// From specific indices
FROM index1, index2, index3

// With metadata
FROM logs-* METADATA _id, _index
```

### ROW - Generate Inline Data
```esql
// Create a single row
ROW name = "John", age = 30, city = "NYC"

// Multiple rows
ROW a = 1, b = "x"
| EVAL c = a * 10
```

---

## Processing Commands

### KEEP - Select Specific Fields
```esql
FROM employees
| KEEP name, department, salary

// Keep with pattern
FROM logs-*
| KEEP @timestamp, message, host.*
```

### DROP - Remove Specific Fields
```esql
FROM employees
| DROP password, ssn, internal_notes

// Drop with pattern
FROM logs-*
| DROP *.keyword
```

### RENAME - Rename Fields
```esql
FROM employees
| RENAME emp_name AS name, emp_dept AS department

// Multiple renames
FROM logs-*
| RENAME source.ip AS src_ip, destination.ip AS dst_ip
```

---

## Data Selection & Filtering

### WHERE - Filter Rows
```esql
// Equality
FROM employees
| WHERE department == "Sales"

// Comparison operators
FROM products
| WHERE price > 100 AND stock < 50

// IS NULL / IS NOT NULL
FROM logs-*
| WHERE error_code IS NOT NULL

// IN operator
FROM employees
| WHERE department IN ("Sales", "Marketing", "HR")

// LIKE operator (wildcards)
FROM logs-*
| WHERE message LIKE "*error*"

// RLIKE operator (regex)
FROM logs-*
| WHERE message RLIKE "error|exception|failure"

// NOT operator
FROM employees
| WHERE NOT department == "IT"

// Multiple conditions
FROM orders
| WHERE status == "completed" 
  AND total_amount > 1000 
  AND order_date >= "2024-01-01"
```

---

## Aggregations & Statistics

### STATS - Aggregate Functions

#### COUNT
```esql
// Count all rows
FROM logs-*
| STATS count = COUNT()

// Count distinct
FROM employees
| STATS unique_departments = COUNT_DISTINCT(department)

// Count by group
FROM logs-*
| STATS event_count = COUNT() BY log_level
```

#### SUM, AVG, MIN, MAX
```esql
FROM sales
| STATS 
    total_revenue = SUM(amount),
    avg_sale = AVG(amount),
    min_sale = MIN(amount),
    max_sale = MAX(amount)

// With grouping
FROM sales
| STATS 
    total = SUM(amount),
    average = AVG(amount)
  BY product_category
```

#### MEDIAN, PERCENTILE
```esql
FROM response_times
| STATS 
    median_time = MEDIAN(duration),
    p95 = PERCENTILE(duration, 95),
    p99 = PERCENTILE(duration, 99)
```

#### Multiple Aggregations
```esql
FROM orders
| STATS 
    order_count = COUNT(),
    total_revenue = SUM(amount),
    avg_order_value = AVG(amount),
    unique_customers = COUNT_DISTINCT(customer_id)
  BY region, product_category
```

---

## String Functions

### CONCAT - Concatenate Strings
```esql
FROM employees
| EVAL full_name = CONCAT(first_name, " ", last_name)

// With separator
FROM logs-*
| EVAL log_info = CONCAT(level, ": ", message)
```

### SUBSTRING - Extract Substring
```esql
FROM employees
| EVAL first_initial = SUBSTRING(first_name, 0, 1)

// Extract with length
FROM products
| EVAL short_code = SUBSTRING(product_id, 0, 5)
```

### LENGTH - String Length
```esql
FROM messages
| EVAL message_length = LENGTH(message)
| WHERE message_length > 100
```

### TRIM, LTRIM, RTRIM - Remove Whitespace
```esql
FROM user_input
| EVAL cleaned = TRIM(input_field)
| EVAL left_trimmed = LTRIM(input_field)
| EVAL right_trimmed = RTRIM(input_field)
```

### UPPER, LOWER - Case Conversion
```esql
FROM employees
| EVAL name_upper = UPPER(name)
| EVAL email_lower = LOWER(email)
```

### REPLACE - Replace String
```esql
FROM logs-*
| EVAL cleaned_message = REPLACE(message, "ERROR", "Warning")
```

### SPLIT - Split String into Array
```esql
FROM logs-*
| EVAL tags_array = SPLIT(tags, ",")
```

### STARTS_WITH, ENDS_WITH
```esql
FROM files
| WHERE STARTS_WITH(filename, "log_")
| WHERE ENDS_WITH(filename, ".txt")
```

---

## Mathematical Functions

### Basic Operations
```esql
FROM sales
| EVAL 
    total = price * quantity,
    discount_price = price * 0.9,
    tax = price * 0.08

// Multiple operations
FROM metrics
| EVAL 
    sum_val = field1 + field2,
    diff_val = field1 - field2,
    product = field1 * field2,
    ratio = field1 / field2,
    remainder = field1 % field2
```

### ABS - Absolute Value
```esql
FROM transactions
| EVAL abs_amount = ABS(transaction_amount)
```

### ROUND, FLOOR, CEIL
```esql
FROM measurements
| EVAL 
    rounded = ROUND(value, 2),
    floored = FLOOR(value),
    ceiled = CEIL(value)
```

### POW - Power
```esql
FROM data
| EVAL squared = POW(value, 2)
| EVAL cubed = POW(value, 3)
```

### SQRT - Square Root
```esql
FROM measurements
| EVAL sqrt_value = SQRT(value)
```

### LOG, LOG10
```esql
FROM data
| EVAL 
    natural_log = LOG(value),
    log_base_10 = LOG10(value)
```

### GREATEST, LEAST
```esql
FROM comparisons
| EVAL 
    max_val = GREATEST(val1, val2, val3),
    min_val = LEAST(val1, val2, val3)
```

---

## Date/Time Functions

### NOW - Current Timestamp
```esql
FROM logs-*
| EVAL current_time = NOW()
```

### DATE_EXTRACT - Extract Date Parts
```esql
FROM events
| EVAL 
    year = DATE_EXTRACT("year", @timestamp),
    month = DATE_EXTRACT("month", @timestamp),
    day = DATE_EXTRACT("day", @timestamp),
    hour = DATE_EXTRACT("hour", @timestamp),
    minute = DATE_EXTRACT("minute", @timestamp),
    day_of_week = DATE_EXTRACT("day_of_week", @timestamp)
```

### DATE_FORMAT - Format Date
```esql
FROM events
| EVAL formatted_date = DATE_FORMAT("yyyy-MM-dd", @timestamp)
| EVAL custom_format = DATE_FORMAT("MMM dd, yyyy HH:mm", @timestamp)
```

### DATE_TRUNC - Truncate Date
```esql
FROM logs-*
| EVAL 
    hour_bucket = DATE_TRUNC("hour", @timestamp),
    day_bucket = DATE_TRUNC("day", @timestamp),
    month_bucket = DATE_TRUNC("month", @timestamp)
```

### DATE_DIFF - Date Difference
```esql
FROM orders
| EVAL days_since_order = DATE_DIFF("days", order_date, NOW())
| EVAL hours_to_delivery = DATE_DIFF("hours", order_date, delivery_date)
```

### DATE_PARSE - Parse String to Date
```esql
FROM data
| EVAL parsed_date = DATE_PARSE("yyyy-MM-dd", date_string)
```

---

## Type Conversion Functions

### TO_STRING - Convert to String
```esql
FROM data
| EVAL id_string = TO_STRING(id)
| EVAL amount_string = TO_STRING(amount)
```

### TO_INTEGER, TO_LONG - Convert to Integer
```esql
FROM data
| EVAL age_int = TO_INTEGER(age_string)
| EVAL id_long = TO_LONG(id_string)
```

### TO_DOUBLE - Convert to Double
```esql
FROM data
| EVAL price_double = TO_DOUBLE(price_string)
```

### TO_BOOLEAN - Convert to Boolean
```esql
FROM data
| EVAL is_active = TO_BOOLEAN(active_string)
```

### TO_DATETIME - Convert to DateTime
```esql
FROM data
| EVAL timestamp = TO_DATETIME(date_string)
```

### TO_IP - Convert to IP Address
```esql
FROM logs-*
| EVAL ip_address = TO_IP(ip_string)
```

---

## Conditional Functions

### CASE - Conditional Logic
```esql
FROM employees
| EVAL salary_grade = CASE(
    salary < 50000, "Entry",
    salary < 80000, "Mid",
    salary < 120000, "Senior",
    "Executive"
  )

// With multiple conditions
FROM orders
| EVAL order_status = CASE(
    status == "pending" AND days_old > 7, "Overdue",
    status == "pending", "Processing",
    status == "shipped", "In Transit",
    status == "delivered", "Completed",
    "Unknown"
  )
```

### COALESCE - Return First Non-Null Value
```esql
FROM data
| EVAL display_name = COALESCE(nickname, first_name, username, "Unknown")
```

### IF - Simple Conditional
```esql
FROM products
| EVAL stock_status = 
    CASE(stock > 0, "Available", "Out of Stock")

// Nested conditions
FROM employees
| EVAL bonus = CASE(
    performance_rating >= 4.5, salary * 0.15,
    performance_rating >= 3.5, salary * 0.10,
    performance_rating >= 2.5, salary * 0.05,
    0
  )
```

---

## Array Functions

### MV_COUNT - Count Array Elements
```esql
FROM logs-*
| EVAL tag_count = MV_COUNT(tags)
| WHERE tag_count > 3
```

### MV_AVG, MV_SUM, MV_MIN, MV_MAX - Array Aggregations
```esql
FROM metrics
| EVAL 
    avg_value = MV_AVG(values),
    total = MV_SUM(values),
    min_value = MV_MIN(values),
    max_value = MV_MAX(values)
```

### MV_CONCAT - Concatenate Array Elements
```esql
FROM logs-*
| EVAL all_tags = MV_CONCAT(tags, ", ")
```

### MV_DEDUPE - Remove Duplicates from Array
```esql
FROM data
| EVAL unique_values = MV_DEDUPE(values)
```

### MV_FIRST, MV_LAST - Get First/Last Element
```esql
FROM logs-*
| EVAL first_tag = MV_FIRST(tags)
| EVAL last_tag = MV_LAST(tags)
```

### MV_SLICE - Extract Array Slice
```esql
FROM data
| EVAL first_three = MV_SLICE(values, 0, 3)
```

---

## Join Operations (LOOKUP)

ES|QL uses ENRICH (similar to LOOKUP/JOIN) to join data from enrich policies.

### Prerequisites: Create Enrich Policy
First, create an enrich policy in Kibana Dev Tools:

```json
PUT /_enrich/policy/user_lookup
{
  "match": {
    "indices": "users",
    "match_field": "user_id",
    "enrich_fields": ["username", "email", "department"]
  }
}

POST /_enrich/policy/user_lookup/_execute
```

### ENRICH - Join/Lookup Data
```esql
FROM logs-*
| ENRICH user_lookup ON user_id
| KEEP @timestamp, user_id, username, email, message

// With field renaming
FROM transactions
| ENRICH product_lookup ON product_id WITH product_name, category, price
| KEEP transaction_id, product_name, category, quantity, price

// Multiple enrichments
FROM orders
| ENRICH customer_lookup ON customer_id WITH customer_name, customer_tier
| ENRICH product_lookup ON product_id WITH product_name, product_category
| KEEP order_id, customer_name, product_name, order_amount
```

### Complex Join Example
```esql
FROM orders
| ENRICH customer_lookup ON customer_id 
    WITH customer_name, customer_email, customer_segment
| ENRICH product_lookup ON product_id 
    WITH product_name, product_category, product_price
| EVAL total_price = quantity * product_price
| WHERE customer_segment == "Premium"
| STATS 
    total_orders = COUNT(),
    total_revenue = SUM(total_price)
  BY customer_name, product_category
| SORT total_revenue DESC
```

---

## Sorting & Limiting

### SORT - Order Results
```esql
// Ascending order (default)
FROM employees
| SORT salary

// Descending order
FROM employees
| SORT salary DESC

// Multiple fields
FROM employees
| SORT department ASC, salary DESC

// With nulls first/last
FROM data
| SORT value DESC NULLS FIRST
```

### LIMIT - Limit Results
```esql
// Get first 10 rows
FROM logs-*
| LIMIT 10

// Top 5 highest salaries
FROM employees
| SORT salary DESC
| LIMIT 5

// Pagination (skip and limit)
FROM products
| SORT price
| LIMIT 20  // Results 0-19
```

### HEAD - Get First N Rows (Alias for LIMIT)
```esql
FROM logs-*
| HEAD 100
```

---

## Grouping & Aggregating

### GROUP BY with STATS
```esql
// Single field grouping
FROM sales
| STATS total_sales = SUM(amount) BY region

// Multiple field grouping
FROM orders
| STATS 
    order_count = COUNT(),
    total_revenue = SUM(amount)
  BY region, product_category, sales_rep

// Time-based grouping
FROM logs-*
| EVAL hour = DATE_TRUNC("hour", @timestamp)
| STATS event_count = COUNT() BY hour, log_level
| SORT hour DESC
```

### Complex Aggregation Example
```esql
FROM sales_data
| WHERE order_date >= "2024-01-01"
| EVAL month = DATE_TRUNC("month", order_date)
| STATS 
    total_orders = COUNT(),
    total_revenue = SUM(amount),
    avg_order_value = AVG(amount),
    unique_customers = COUNT_DISTINCT(customer_id),
    max_order = MAX(amount),
    min_order = MIN(amount)
  BY month, region, product_category
| EVAL revenue_per_customer = total_revenue / unique_customers
| WHERE total_orders > 100
| SORT month DESC, total_revenue DESC
| LIMIT 50
```

---

## Advanced Patterns

### Window Functions Pattern
```esql
// Running total by group
FROM sales
| SORT date
| STATS 
    daily_sales = SUM(amount),
    order_count = COUNT()
  BY date, region
| SORT region, date
```

### Pivoting Data
```esql
// Count by status and priority
FROM tickets
| STATS ticket_count = COUNT() BY status, priority
| SORT status, priority
```

### Finding Duplicates
```esql
FROM users
| STATS count = COUNT() BY email
| WHERE count > 1
| SORT count DESC
```

### Time Series Analysis
```esql
FROM metrics-*
| EVAL 
    hour = DATE_TRUNC("hour", @timestamp),
    day = DATE_EXTRACT("day", @timestamp)
| STATS 
    avg_cpu = AVG(cpu_percent),
    max_cpu = MAX(cpu_percent),
    avg_memory = AVG(memory_percent)
  BY hour, host
| WHERE avg_cpu > 80
| SORT hour DESC
```

### Percentage Calculations
```esql
FROM sales
| STATS 
    total_sales = SUM(amount),
    count = COUNT()
  BY product_category
| EVAL percentage = ROUND(total_sales / SUM(total_sales) * 100, 2)
| SORT percentage DESC
```

### Top N per Group
```esql
// Top 3 products per category by sales
FROM sales
| STATS total_sales = SUM(amount) BY product_category, product_name
| SORT product_category, total_sales DESC
// Note: ES|QL doesn't have native PARTITION BY, 
// so you may need to process this in multiple queries or use aggregations
```

### Data Cleaning
```esql
FROM raw_data
| EVAL 
    // Clean whitespace
    cleaned_name = TRIM(name),
    // Standardize case
    email_lower = LOWER(email),
    // Replace values
    status = REPLACE(status, "N/A", "Unknown"),
    // Handle nulls
    age = COALESCE(age, 0),
    // Validate ranges
    valid_age = CASE(age < 0 OR age > 150, NULL, age)
| WHERE cleaned_name IS NOT NULL
| DROP name, email
| RENAME cleaned_name AS name, email_lower AS email
```

### Cohort Analysis
```esql
FROM user_events
| EVAL 
    signup_month = DATE_TRUNC("month", signup_date),
    event_month = DATE_TRUNC("month", event_date)
| STATS 
    active_users = COUNT_DISTINCT(user_id)
  BY signup_month, event_month
| SORT signup_month, event_month
```

### Anomaly Detection Pattern
```esql
FROM metrics-*
| EVAL hour = DATE_TRUNC("hour", @timestamp)
| STATS 
    avg_value = AVG(value),
    stddev = SQRT(AVG(POW(value - AVG(value), 2)))
  BY hour
| EVAL 
    upper_bound = avg_value + (2 * stddev),
    lower_bound = avg_value - (2 * stddev)
```

---

## Complete Real-World Examples

### Example 1: User Activity Dashboard
```esql
FROM user_logs-*
| WHERE @timestamp >= NOW() - 7 days
| ENRICH user_lookup ON user_id WITH username, user_tier
| EVAL day = DATE_TRUNC("day", @timestamp)
| STATS 
    daily_active_users = COUNT_DISTINCT(user_id),
    total_sessions = COUNT(),
    avg_session_duration = AVG(session_duration)
  BY day, user_tier
| EVAL avg_duration_minutes = ROUND(avg_session_duration / 60, 2)
| SORT day DESC, user_tier
```

### Example 2: E-commerce Sales Report
```esql
FROM orders
| WHERE order_date >= "2024-01-01"
| ENRICH customer_lookup ON customer_id 
    WITH customer_name, customer_segment
| ENRICH product_lookup ON product_id 
    WITH product_name, product_category, cost_price
| EVAL 
    profit = (price - cost_price) * quantity,
    month = DATE_TRUNC("month", order_date)
| STATS 
    total_orders = COUNT(),
    total_revenue = SUM(price * quantity),
    total_profit = SUM(profit),
    avg_order_value = AVG(price * quantity),
    unique_customers = COUNT_DISTINCT(customer_id)
  BY month, product_category, customer_segment
| EVAL profit_margin = ROUND(total_profit / total_revenue * 100, 2)
| WHERE total_revenue > 10000
| SORT month DESC, total_revenue DESC
| LIMIT 100
```

### Example 3: Security Log Analysis
```esql
FROM security-logs-*
| WHERE @timestamp >= NOW() - 24 hours
| WHERE event_type IN ("login_failed", "suspicious_activity")
| EVAL hour = DATE_TRUNC("hour", @timestamp)
| STATS 
    event_count = COUNT(),
    unique_ips = COUNT_DISTINCT(source_ip),
    unique_users = COUNT_DISTINCT(username)
  BY hour, event_type, country
| WHERE event_count > 100
| SORT hour DESC, event_count DESC
```

### Example 4: Application Performance Monitoring
```esql
FROM apm-*
| WHERE @timestamp >= NOW() - 1 hour
| EVAL 
    response_category = CASE(
        response_time < 100, "Fast",
        response_time < 500, "Medium",
        response_time < 1000, "Slow",
        "Very Slow"
    ),
    minute = DATE_TRUNC("minute", @timestamp)
| STATS 
    request_count = COUNT(),
    avg_response = AVG(response_time),
    p95_response = PERCENTILE(response_time, 95),
    p99_response = PERCENTILE(response_time, 99),
    error_count = COUNT() WHERE status_code >= 400
  BY minute, endpoint, response_category
| EVAL error_rate = ROUND(error_count / request_count * 100, 2)
| WHERE request_count > 10
| SORT minute DESC, avg_response DESC
```

---

## Tips & Best Practices

1. **Use KEEP instead of SELECT** - More explicit about which fields to retain
2. **Filter early with WHERE** - Reduce data processing by filtering before aggregations
3. **Use DATE_TRUNC for time bucketing** - Essential for time series analysis
4. **Leverage ENRICH for joins** - Pre-create enrich policies for frequently joined data
5. **Use EVAL for calculated fields** - Create derived fields before aggregation
6. **Combine multiple conditions in WHERE** - More efficient than multiple WHERE clauses
7. **Use STATS with BY for grouping** - Replaces traditional GROUP BY
8. **Sort after aggregation** - More efficient than sorting before
9. **Use LIMIT to control output size** - Especially important for large datasets
10. **Use metadata fields when needed** - Access _id, _index with METADATA keyword

---

## Common Patterns Cheat Sheet

```esql
// Count by field
FROM index | STATS count = COUNT() BY field

// Top N
FROM index | STATS value = SUM(amount) BY category | SORT value DESC | LIMIT 10

// Time series
FROM index | EVAL bucket = DATE_TRUNC("hour", @timestamp) | STATS count = COUNT() BY bucket

// Percentage of total
FROM index | STATS total = SUM(amount) BY category | EVAL pct = total / SUM(total) * 100

// Filter nulls
FROM index | WHERE field IS NOT NULL

// String matching
FROM index | WHERE field LIKE "*pattern*"

// Date range
FROM index | WHERE @timestamp >= NOW() - 7 days

// Multiple aggregations
FROM index | STATS count = COUNT(), sum = SUM(val), avg = AVG(val) BY group

// Conditional aggregation
FROM index | STATS error_count = COUNT() WHERE status == "error" BY service
```

---

## Comparison with Traditional SQL

| SQL | ES|QL |
|-----|-------|
| SELECT * | FROM index |
| SELECT field1, field2 | FROM index \| KEEP field1, field2 |
| WHERE condition | WHERE condition (same) |
| GROUP BY field | STATS ... BY field |
| ORDER BY field | SORT field |
| LIMIT 10 | LIMIT 10 (same) |
| COUNT(*) | STATS count = COUNT() |
| SUM(field) | STATS total = SUM(field) |
| AVG(field) | STATS avg = AVG(field) |
| JOIN | ENRICH (using enrich policies) |
| CASE WHEN | CASE(...) |
| CONCAT(a, b) | CONCAT(a, b) (same) |

---

## Resources

- Official ES|QL Documentation: https://www.elastic.co/guide/en/elasticsearch/reference/current/esql.html
- ES|QL Functions Reference: https://www.elastic.co/guide/en/elasticsearch/reference/current/esql-functions.html
- Enrich Processor: https://www.elastic.co/guide/en/elasticsearch/reference/current/enrich-processor.html

---

*Last Updated: 2024*
*ES|QL is actively evolving - check official documentation for latest features*
