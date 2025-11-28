# Northwind SQL Analytics Project

🇧🇷 **[Versão em Português](README_PT.md)**

## About This Project

This repository contains comprehensive SQL analytics for the Northwind database. The project demonstrates advanced SQL techniques and data analysis methods that can be applied to real business scenarios. These reports help organizations extract actionable insights from their data for better strategic decision-making.

**Author:** [yagosamu](https://github.com/yagosamu)

## SQL Reports Overview

This project includes the following analytical reports:

1. **Revenue Reports**
    
    * What was the total revenue in 1997?

    ```sql
    CREATE VIEW total_revenues_1997_view AS
    SELECT SUM((order_details.unit_price) * order_details.quantity * (1.0 - order_details.discount)) AS total_revenues_1997
    FROM order_details
    INNER JOIN (
        SELECT order_id 
        FROM orders 
        WHERE EXTRACT(YEAR FROM order_date) = '1997'
    ) AS ord 
    ON ord.order_id = order_details.order_id;
    ```

    * Perform monthly growth analysis and YTD calculation

    ```sql
    CREATE VIEW view_receitas_acumuladas AS
    WITH ReceitasMensais AS (
        SELECT
            EXTRACT(YEAR FROM orders.order_date) AS Ano,
            EXTRACT(MONTH FROM orders.order_date) AS Mes,
            SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS Receita_Mensal
        FROM
            orders
        INNER JOIN
            order_details ON orders.order_id = order_details.order_id
        GROUP BY
            EXTRACT(YEAR FROM orders.order_date),
            EXTRACT(MONTH FROM orders.order_date)
    ),
    ReceitasAcumuladas AS (
        SELECT
            Ano,
            Mes,
            Receita_Mensal,
            SUM(Receita_Mensal) OVER (PARTITION BY Ano ORDER BY Mes) AS Receita_YTD
        FROM
            ReceitasMensais
    )
    SELECT
        Ano,
        Mes,
        Receita_Mensal,
        Receita_Mensal - LAG(Receita_Mensal) OVER (PARTITION BY Ano ORDER BY Mes) AS Diferenca_Mensal,
        Receita_YTD,
        (Receita_Mensal - LAG(Receita_Mensal) OVER (PARTITION BY Ano ORDER BY Mes)) / LAG(Receita_Mensal) OVER (PARTITION BY Ano ORDER BY Mes) * 100 AS Percentual_Mudanca_Mensal
    FROM
        ReceitasAcumuladas
    ORDER BY
        Ano, Mes;
    ```

2. **Customer Segmentation**
    
    * What is the total amount each customer has paid so far?

    ```sql
    CREATE VIEW view_total_revenues_per_customer AS
    SELECT 
        customers.company_name, 
        SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS total
    FROM 
        customers
    INNER JOIN 
        orders ON customers.customer_id = orders.customer_id
    INNER JOIN 
        order_details ON order_details.order_id = orders.order_id
    GROUP BY 
        customers.company_name
    ORDER BY 
        total DESC;
    ```

    * Separate customers into 5 groups according to the amount paid per customer

    ```sql
    CREATE VIEW view_total_revenues_per_customer_group AS
    SELECT 
    customers.company_name, 
    SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS total,
    NTILE(5) OVER (ORDER BY SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) DESC) AS group_number
    FROM 
        customers
    INNER JOIN 
        orders ON customers.customer_id = orders.customer_id
    INNER JOIN 
        order_details ON order_details.order_id = orders.order_id
    GROUP BY 
        customers.company_name
    ORDER BY 
        total DESC;
    ```


    * Now only customers who are in groups 3, 4 and 5 so that a special Marketing analysis can be done with them

    ```sql
    CREATE VIEW clients_to_marketing AS
    WITH clientes_para_marketing AS (
        SELECT 
        customers.company_name, 
        SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS total,
        NTILE(5) OVER (ORDER BY SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) DESC) AS group_number
    FROM 
        customers
    INNER JOIN 
        orders ON customers.customer_id = orders.customer_id
    INNER JOIN 
        order_details ON order_details.order_id = orders.order_id
    GROUP BY 
        customers.company_name
    ORDER BY 
        total DESC
    )

    SELECT *
    FROM clientes_para_marketing
    WHERE group_number >= 3;
    ```

3. **Top 10 Best Selling Products**
    
    * Identify the 10 best selling products.

    ```sql
    CREATE VIEW top_10_products AS
    SELECT products.product_name, SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) AS sales
    FROM products
    INNER JOIN order_details ON order_details.product_id = products.product_id
    GROUP BY products.product_name
    ORDER BY sales DESC;
    ```

4. **UK Customers Who Paid More Than $1000**
    
    * Which customers in the UK paid more than $1000?

    ```sql
    CREATE VIEW uk_clients_who_pay_more_then_1000 AS
    SELECT customers.contact_name, SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount) * 100) / 100 AS payments
    FROM customers
    INNER JOIN orders ON orders.customer_id = customers.customer_id
    INNER JOIN order_details ON order_details.order_id = orders.order_id
    WHERE LOWER(customers.country) = 'uk'
    GROUP BY customers.contact_name
    HAVING SUM(order_details.unit_price * order_details.quantity * (1.0 - order_details.discount)) > 1000;
    ```

## Context

The `Northwind` database contains sales data from a company called `Northwind Traders`, which imports and exports special foods from around the world.

The Northwind database is an ERP with customer, order, inventory, purchasing, supplier, shipping, employee and accounting data.

The Northwind dataset includes sample data for the following:

* **Suppliers:** Northwind suppliers and vendors
* **Customers:** Customers who buy products from Northwind
* **Employees:** Details of Northwind Traders employees
* **Products:** Product information
* **Shippers:** Details of shippers who ship products from merchants to end customers
* **Orders and Order Details:** Sales order transactions occurring between customers and the company

The `Northwind` database includes 14 tables and the relationships between the tables are shown in the following entity relationship diagram.

![northwind](pics/northwind-er-diagram.png)

## Initial Setup

### Manual

Use the provided SQL file, `northwind.sql`, to populate your database.

### With Docker and Docker Compose

**Prerequisites**: Install Docker and Docker Compose

* [Get started with Docker](https://www.docker.com/get-started)
* [Install Docker Compose](https://docs.docker.com/compose/install/)

### Steps for Docker setup:

1. **Start Docker Compose** Run the command below to start the services:
    
    ```
    docker-compose up
    ```
    
    Wait for the configuration messages, such as:
    
    ```csharp
    Creating network "northwind_psql_db" with driver "bridge"
    Creating volume "northwind_psql_db" with default driver
    Creating volume "northwind_psql_pgadmin" with default driver
    Creating pgadmin ... done
    Creating db      ... done
    ```
       
2. **Connect PgAdmin** Access PgAdmin at URL: [http://localhost:5050](http://localhost:5050), with password `postgres`. 

Configure a new server in PgAdmin:
    
    * **General Tab**:
        * Name: db
    * **Connection Tab**:
        * Host name: db
        * Username: postgres
        * Password: postgres Then select the "northwind" database.

3. **Stop Docker Compose** Stop the server started by the `docker-compose up` command using Ctrl-C and remove containers with:
    
    ```
    docker-compose down
    ```
    
4. **Files and Persistence** Your Postgres database modifications will be persisted in the Docker volume `postgresql_data` and can be recovered by restarting Docker Compose with `docker-compose up`. To delete database data, run:
    
    ```
    docker-compose down -v
    ```