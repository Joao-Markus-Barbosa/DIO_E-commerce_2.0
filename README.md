# DIO_E-commerce_2.0
# Sistema de Gestão de Pedidos e Produtos

Este projeto implementa um sistema para gerenciar pedidos, clientes, produtos, fornecedores, transportadoras e métodos de pagamento. O sistema permite realizar operações de CRUD nas tabelas do banco de dados, além de realizar consultas e gerar relatórios úteis para o gerenciamento da plataforma.

## Estrutura do Banco de Dados

O banco de dados do sistema possui as seguintes tabelas:

### Tabelas Principais

- **T_CLIENT**: Contém informações sobre os clientes da plataforma (Pessoa Física ou Jurídica).
- **T_PRODUCT**: Contém informações sobre os produtos disponíveis para venda.
- **T_ORDER**: Contém informações sobre os pedidos realizados pelos clientes.
- **T_STOCK**: Contém informações sobre o estoque de produtos.
- **T_SUPPLIER**: Contém informações sobre os fornecedores de produtos.
- **T_SELLER**: Contém informações sobre os vendedores cadastrados.
- **T_TRANSPORT**: Contém informações sobre as transportadoras utilizadas para a entrega dos produtos.
- **T_BANK**: Contém informações sobre os bancos utilizados para o processamento de pagamentos.
- **T_PAYMENT_METHOD**: Contém os diferentes métodos de pagamento disponíveis na plataforma.

### Tabelas de Relacionamento (N:M)

- **T_PAYMENT**: Registra os pagamentos realizados para os pedidos.
- **T_DELIVERY**: Contém informações sobre as entregas realizadas pelos transportadores.
- **T_PRODUCT_STOCK**: Relaciona os produtos aos estoques em que estão armazenados.
- **T_STOCK_SUPPLIER**: Relaciona os fornecedores aos estoques.
- **T_PRODUCT_SELLER**: Relaciona os vendedores aos produtos que estão vendendo.
- **T_ORDER_PRODUCT**: Relaciona os pedidos aos produtos vendidos.

## Funcionalidades

O sistema permite realizar diversas consultas e operações, como:

### Consultas Simples
1. **Recuperação de Dados de Clientes**
    - Recupera os dados dos clientes cadastrados na plataforma.
    - Exemplo de consulta:
    ```sql
    SELECT 
        id_Client, 
        client_Name, 
        client_Type, 
        client_CPF, 
        client_CNPJ, 
        client_Address 
    FROM 
        T_CLIENT;
    ```

### Filtros com WHERE
2. **Busca de Pedidos Confirmados de Clientes do Tipo "PJ"**
    - Filtra os pedidos confirmados realizados por clientes do tipo "Pessoa Jurídica".
    - Exemplo de consulta:
    ```sql
    SELECT 
        o.id_Order, 
        c.client_Name, 
        o.order_Status, 
        o.order_Value
    FROM 
        T_ORDER o
    JOIN 
        T_CLIENT c ON o.id_Order_Client = c.id_Client
    WHERE 
        o.order_Status = 'Confirmado' 
        AND c.client_Type = 'PJ';
    ```

### Expressões Derivadas
3. **Cálculo de Valor Total de Pedido**
    - Calcula o valor total a ser pago pelo cliente, incluindo o pedido e o frete.
    - Exemplo de consulta:
    ```sql
    SELECT 
        o.id_Order, 
        c.client_Name, 
        o.order_Value, 
        o.freight_value, 
        (o.order_Value + o.freight_value) AS total_Payment, 
        CURRENT_DATE - INTERVAL 7 DAY AS last_Week_Date 
    FROM 
        T_ORDER o
    JOIN 
        T_CLIENT c ON o.id_Order_Client = c.id_Client;
    ```

### Ordenação de Dados
4. **Ordenação de Pedidos pelo Valor Total**
    - Ordena os pedidos pelo valor total (pedido + frete) de forma decrescente.
    - Exemplo de consulta:
    ```sql
    SELECT 
        o.id_Order, 
        c.client_Name, 
        o.order_Value, 
        o.freight_value, 
        (o.order_Value + o.freight_value) AS total_Payment
    FROM 
        T_ORDER o
    JOIN 
        T_CLIENT c ON o.id_Order_Client = c.id_Client
    ORDER BY 
        total_Payment DESC;
    ```

### Filtros com HAVING
5. **Filtragem de Clientes que Gastaram Acima de 1000**
    - Conta quantos pedidos cada cliente fez e filtra os clientes que gastaram mais de 1000 no total.
    - Exemplo de consulta:
    ```sql
    SELECT 
        c.client_Name, 
        COUNT(o.id_Order) AS total_Orders, 
        SUM(o.order_Value) AS total_Spent
    FROM 
        T_ORDER o
    JOIN 
        T_CLIENT c ON o.id_Order_Client = c.id_Client
    GROUP BY 
        c.client_Name
    HAVING 
        SUM(o.order_Value) > 1000;
    ```

### Junções entre Tabelas
6. **Identificar Produtos Mais Vendidos e Receita Total**
    - Identifica quais produtos foram mais vendidos e quanto faturaram no total.
    - Exemplo de consulta:
    ```sql
    SELECT 
        p.product_Name AS produto,
        SUM(op.PO_Quantity) AS total_vendido,
        ROUND(SUM(op.PO_Quantity * p.product_Value), 2) AS receita_total
    FROM 
        T_ORDER_PRODUCT op
    INNER JOIN 
        T_PRODUCT p ON op.id_OP_Product = p.id_Product
    INNER JOIN 
        T_ORDER o ON op.id_OP_Order = o.id_Order
    WHERE 
        o.order_Status = 'Confirmado'
    GROUP BY 
        p.product_Name
    ORDER BY 
        receita_total DESC
    LIMIT 10;
    ```
