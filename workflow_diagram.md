flowchart TB

    Customer["Customer"]

    subgraph Supervisor["CustomerOperationsSupervisor (orchestrator_agent)"]
        direction TB
        Route{"Routing Rules<br/>Keyword-Based Prompt"}
    end

    subgraph Inventory["InventorySpecialist"]

        IT1["get_inventory_status<br/>Purpose: retrieve stock level<br/>Uses: resolve_inventory_item_name,<br/>get_stock_level"]

        IT2["check_inventory_availability<br/>Purpose: validate requested quantity<br/>Uses: resolve_inventory_item_name,<br/>get_stock_level"]

        IT3["get_delivery_timeline<br/>Purpose: estimate delivery date<br/>Uses: get_supplier_delivery_date"]

        IT4["get_all_available_inventory<br/>Purpose: inventory snapshot<br/>Uses: get_all_inventory"]

    end

    subgraph Quote["QuoteSpecialistAgent"]

        QT1["search_quote_history<br/>Purpose: find similar past quotes<br/>Uses: quotes table,<br/>quote_requests table"]

        QT2["calculate_quote<br/>Purpose: calculate quote amount<br/>Uses: item_price × quantity"]

    end

    subgraph Sales["SalesTransactionAgent"]

        ST1["can_fulfill_order<br/>Purpose: validate customer order<br/>Uses: resolve_inventory_item_name,<br/>get_stock_level"]

        ST2["create_sales_order<br/>Purpose: create sale transaction<br/>Uses: resolve_inventory_item_name,<br/>get_stock_level,<br/>get_unit_price_for_item,<br/>create_transaction,<br/>get_supplier_delivery_date"]

        ST3["get_all_available_inventory<br/>Purpose: inventory snapshot<br/>Uses: get_all_inventory"]

        ST4["get_delivery_timeline<br/>Purpose: estimate delivery date<br/>Uses: get_supplier_delivery_date"]

        ST5["get_request_date<br/>Purpose: extract request date<br/>Uses: regex extraction"]

    end

    subgraph Supply["SupplyOperationsSpecialist"]

        SU1["check_reorder_needed<br/>Purpose: compare stock to minimum level<br/>Uses: resolve_inventory_item_name,<br/>get_inventory_metadata,<br/>get_stock_level"]

        SU2["check_cash_available<br/>Purpose: verify cash balance<br/>Uses: get_cash_balance"]

        SU3["create_purchase_order<br/>Purpose: restock inventory<br/>Uses: resolve_inventory_item_name,<br/>get_inventory_metadata,<br/>get_cash_balance,<br/>create_transaction,<br/>get_supplier_delivery_date"]

        SU4["get_financial_report<br/>Purpose: financial reporting<br/>Uses: generate_financial_report"]

        SU5["get_request_date<br/>Purpose: extract request date<br/>Uses: regex extraction"]

    end

    subgraph Helpers["Shared Helper Functions"]

        H1["resolve_inventory_item_name"]

        H2["get_stock_level"]

        H3["get_all_inventory"]

        H4["get_unit_price_for_item"]

        H5["get_inventory_metadata"]

        H6["create_transaction"]

        H7["get_cash_balance"]

        H8["get_supplier_delivery_date"]

        H9["generate_financial_report"]

    end

    DB[("munder_difflin.db<br/>SQLite")]

    Customer --> Supervisor

    Supervisor -- inventory / stock / availability --> Inventory
    Supervisor -- quote / estimate / pricing --> Quote
    Supervisor -- buy / purchase / order --> Sales
    Supervisor -- reorder / supplier / replenishment --> Supply

    Sales -- automatically triggered<br/>after completed sale --> Supply

    Inventory --> DB
    Quote --> DB
    Sales --> DB
    Supply --> DB

    %% Inventory helper mappings
    IT1 -.uses.-> H1
    IT1 -.uses.-> H2

    IT2 -.uses.-> H1
    IT2 -.uses.-> H2

    IT3 -.uses.-> H8

    IT4 -.uses.-> H3

    %% Sales helper mappings
    ST1 -.uses.-> H1
    ST1 -.uses.-> H2

    ST2 -.uses.-> H1
    ST2 -.uses.-> H2
    ST2 -.uses.-> H4
    ST2 -.uses.-> H6
    ST2 -.uses.-> H8

    ST3 -.uses.-> H3

    ST4 -.uses.-> H8

    %% Supply helper mappings
    SU1 -.uses.-> H1
    SU1 -.uses.-> H5
    SU1 -.uses.-> H2

    SU2 -.uses.-> H7

    SU3 -.uses.-> H1
    SU3 -.uses.-> H5
    SU3 -.uses.-> H7
    SU3 -.uses.-> H6
    SU3 -.uses.-> H8

    SU4 -.uses.-> H9

    %% Helper to DB mappings
    H2 --> DB
    H3 --> DB
    H5 --> DB
    H6 --> DB
    H7 --> DB
    H9 --> DB
