# Nantinya, table ini akan digunakan untuk sebuah Laporan yang bertujuan untuk menyajikan data penjualan setiap bulan berdasarkan product. Tujuan utama dari laporan ini adalah untuk mengidentifikasi produk dengan penjualan tertinggi setiap bulannya.

# DATA PENJUALAN SETIAP BULAN BERDASARKAN PRODUK
``` sql
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
```
Penjelasan:
1. WITH monthly_sales AS (...): Bagian ini membuat Common Table Expression (CTE) bernama monthly_sales. CTE ini bertindak sebagai tabel sementara yang menyimpan hasil dari query yang ada di dalamnya

   a. SELECT product.name AS product_name, EXTRACT(YEAR FROM order_item.created_at) AS year, EXTRACT(MONTH FROM order_item.created_at) AS month, SUM(order_item.sale_price) AS total_sales
   
   Memilih kolom yang akan ditampilkan. Satu, kolom name di table products (product_name) yang berisi informasi mengenai nama produk. Dua, extract tahun dari kolom created_at di tabel order_items dan menyimpannya sebagai alias year. Tiga, extract bulan dari kolom created_at di tabel order_items dan menyimpannya dengan alias year. Empat, menghitung total penjualan (sale_price) untuk setiap kombinasi produk, tahun, dan bulan dengan alias total_sales).
   
   b. FROM `bigquery-public-data.thelook_ecommerce.products` AS product

   Query ini mengakses data dari tabel products yang berada dalam dataset thelook_ecommerce yang merupakan bagian dari bigquery-public-data.
   
   c. LEFT JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS order_item

   Menggabungkan tabel products dengan tabel order_items di mana kolom id dari products cocok dengan kolom product_id di order_items. Semua baris dari products akan dimasukkan, meskipun tidak ada kecocokan di order_items.
   
   d. ON product.id = order_item.product_id

   Kondisi penggabungan yang menyatakan bahwa penggabungan harus dilakukan di mana id dari products sama dengan product_id di order_items.

   e. GROUP BY product.name, year, month

   Mengelompokkan hasil berdasarkan nama produk, tahun, dan bulan. Ini memastikan bahwa total penjualan dihitung untuk setiap kombinasi unik dari nama produk, tahun, dan bulan.
   
2. SELECT product_name, year, month, total_sales, sales_rank
   
   Memilih kolom yang akan ditampilkan. Satu, product_name dari tabel products. Dua, extract tahun dari kolom created_at di tabel order_items. Tiga, extract bulan dari kolom created_at di tabel order_items. Empat, total penjualan (sale_price) untuk setiap kombinasi produk, tahun, dan bulan dari tabel order_items. Lima, sales_rank yaitu peringkat produk berdasarkan total_sales untuk setiap bulan dan tahun.

3. FROM monthly_sales
   Mengambil data dari CTE monthly_sales.
   
4. ORDER BY year DESC, month DESC, total_sales DESC;

   Order By: Hasilnya diurutkan berdasarkan year, month, dan total_sales secara menurun, sehingga data terbaru dengan penjualan tertinggi ditampilkan terlebih dahulu.



# PRODUK DENGAN PENJUALAN TERTINGGI SETIAP BULAN

``` sql
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
    FROM monthly_sales
)
SELECT
    product_name,
    year,
    month,
    total_sales,
    sales_rank
FROM ranked_sales
WHERE sales_rank = 1
ORDER BY
    year DESC,
    month DESC,
    total_sales DESC;
```
Penjelasan:

1. WITH monthly_sales AS (...): Bagian ini membuat Common Table Expression (CTE) bernama monthly_sales. CTE ini bertindak sebagai tabel sementara yang menyimpan hasil dari query yang ada di dalamnya

   a. SELECT product.name AS product_name, EXTRACT(YEAR FROM order_item.created_at) AS year, EXTRACT(MONTH FROM order_item.created_at) AS month, SUM(order_item.sale_price) AS total_sales
   
   Memilih kolom yang akan ditampilkan. Satu, kolom name di table products (product_name) yang berisi informasi mengenai nama produk. Dua, extract tahun dari kolom created_at di tabel order_items dan menyimpannya sebagai alias year. Tiga, extract bulan dari kolom created_at di tabel order_items dan menyimpannya dengan alias year. Empat, menghitung total penjualan (sale_price) untuk setiap kombinasi produk, tahun, dan bulan dengan alias total_sales).
   
   b. FROM `bigquery-public-data.thelook_ecommerce.products` AS product
   
   Query ini mengakses data dari tabel products yang berada dalam dataset thelook_ecommerce yang merupakan bagian dari bigquery-public-data.
   
   c. LEFT JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS order_item

   Menggabungkan tabel products dengan tabel order_items di mana kolom id dari products cocok dengan kolom product_id di order_items. Semua baris dari products akan dimasukkan, meskipun tidak ada kecocokan di order_items.
   
   d. ON product.id = order_item.product_id

   Kondisi penggabungan yang menyatakan bahwa penggabungan harus dilakukan di mana id dari products sama dengan product_id di order_items.

   e. GROUP BY product.name, year, month

   Mengelompokkan hasil berdasarkan nama produk, tahun, dan bulan. Ini memastikan bahwa total penjualan dihitung untuk setiap kombinasi unik dari nama produk, tahun, dan bulan.
   
2. ranked_sales AS (...): Bagian ini membuat CTE kedua bernama ranked_sales yang akan menyimpan hasil dari query yang mengambil data dari monthly_sales dan memberikan peringkat berdasarkan total penjualan.
   
   a. SELECT product_name, year, month, total_sales, RANK() OVER (PARTITION BY year, month ORDER BY total_sales DESC) AS sales_rank
   
   Memilih kolom yang akan ditampilkan. Satu, product_name dari tabel products. Dua, extract tahun dari kolom created_at di tabel order_items. Tiga, extract bulan dari kolom created_at di tabel order_items. Empat, total penjualan (sale_price) untuk setiap kombinasi produk, tahun, dan bulan dari tabel order_items. Lima, sales_rank, peringkat (RANK) pada produk berdasarkan total penjualan (total_sales) untuk setiap bulan dan tahun. Peringkat ini diberikan dalam urutan menurun (DESC) sehingga produk dengan penjualan tertinggi akan memiliki peringkat 1.

   b. FROM monthly_sales
   
   Mengambil data dari CTE monthly_sales untuk kemudian diproses lebih lanjut dalam CTE ranked_sales.

3. SELECT product_name, year, month, total_sales, sales_rank
   
   Memilih kolom yang akan ditampilkan. Satu, product_name dari tabel products. Dua, extract tahun dari kolom created_at di tabel order_items. Tiga, extract bulan dari kolom created_at di tabel order_items. Empat, total penjualan (sale_price) untuk setiap kombinasi produk, tahun, dan bulan dari tabel order_items. Lima, sales_rank yaitu peringkat produk berdasarkan total_sales untuk setiap bulan dan tahun.

4. FROM ranked_sales
   Mengambil data dari CTE ranked_sales.

5. WHERE sales_rank = 1
   
   filter sales_rank = 1: Hanya memilih baris yang memiliki sales_rank = 1, yang berarti produk dengan penjualan tertinggi di setiap bulan.
   
8. ORDER BY year DESC, month DESC, total_sales DESC;

   Order By: Hasilnya diurutkan berdasarkan year, month, dan total_sales secara menurun, sehingga data terbaru dengan penjualan tertinggi ditampilkan terlebih dahulu.
