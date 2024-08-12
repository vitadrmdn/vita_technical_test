# Buatlah SQL dan temporary table untuk report_monthly_orders_product_agg!
# Nantinya, table ini akan digunakan untuk sebuah Laporan yang bertujuan untuk menyajikan data penjualan setiap bulan berdasarkan product. Tujuan utama dari laporan ini adalah untuk mengidentifikasi produk dengan penjualan tertinggi setiap bulannya.

# Memunculkan seluruh produk
WITH monthly_sales AS (
    SELECT
        product.name AS product_name,
        EXTRACT(YEAR FROM order_item.created_at) AS year,
        EXTRACT(MONTH FROM order_item.created_at) AS month,
        SUM(order_item.sale_price) AS total_sales
    FROM `bigquery-public-data.thelook_ecommerce.products` AS product
    LEFT JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS order_item
    ON product.id = order_item.product_id
    GROUP BY product.name, year, month
)
SELECT
    product_name,
    year,
    month,
    total_sales,
    RANK() OVER (PARTITION BY year, month ORDER BY total_sales DESC) AS sales_rank
FROM
    monthly_sales
ORDER BY
    year DESC,
    month DESC,
    total_sales DESC;

# Memunculkan produk dengan penjualan tertinggi setiap bulannya.
WITH monthly_sales AS (
    SELECT
        product.name AS product_name,
        EXTRACT(YEAR FROM order_item.created_at) AS year,
        EXTRACT(MONTH FROM order_item.created_at) AS month,
        SUM(order_item.sale_price) AS total_sales
    FROM `bigquery-public-data.thelook_ecommerce.products` AS product
    LEFT JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS order_item
    ON product.id = order_item.product_id
    GROUP BY product.name, year, month
),
ranked_sales AS (
    SELECT
        product_name,
        year,
        month,
        total_sales,
        RANK() OVER (PARTITION BY year, month ORDER BY total_sales DESC) AS sales_rank
    FROM
        monthly_sales
)
SELECT
    product_name,
    year,
    month,
    total_sales,
    sales_rank
FROM
    ranked_sales
WHERE
    sales_rank = 1
ORDER BY
    year DESC,
    month DESC,
    total_sales DESC;
