
flowchart TB
 subgraph Supervisor["CustomerOperationsSupervisor (orchestrator_agent)"]
    direction TB
        Route{"Routing rules<br>keyword-based"}
  end
 subgraph Inventory["InventorySpecialist"]
    direction TB
        IT1["get_inventory_status"]
        IT2["check_inventory_availability"]
        IT3["get_delivery_timeline"]
        IT4["get_all_available_inventory"]
  end
 subgraph Quote["QuoteSpecialistAgent"]
    direction TB
        QT1["search_quote_history"]
        QT2["calculate_quote"]
  end
 subgraph Sales["SalesTransactionAgent"]
    direction TB
        ST1["can_fulfill_order"]
        ST2["create_sales_order"]
        ST3["get_all_available_inventory"]
        ST4["get_delivery_timeline"]
        ST5["get_request_date"]
  end
 subgraph Supply["SupplyOperationsSpecialist"]
    direction TB
        SU1["check_reorder_needed"]
        SU2["check_cash_available"]
        SU3["create_purchase_order"]
        SU4["get_request_date"]
  end
    Supervisor -- inventory status,<br>stock, delivery timeline --> Inventory
    Supervisor -- quote, estimate,<br>price, cost --> Quote
    Supervisor -- buy, purchase,<br>order, transaction --> Sales
    Sales -- if sale completed<br>(fully or partially fulfilled) --> Supply
    Inventory --> DB[("munder_difflin.db<br>SQLite")]
    Quote --> DB
    Sales --> DB
    Supply --> DB
    Sales -. sales_response text .-> Result(["Response returned<br>to Supervisor"])
    Supply -. supply_follow_up text .-> Result
    Inventory -.-> Result
    Quote -.-> Result
    Result --> Supervisor
    n1["Customer"] --> Supervisor
    Supervisor --> n1

    n1@{ shape: rect}
    style Supervisor fill:#2563eb22,stroke:#2563eb
    style Inventory fill:#0891b222,stroke:#0891b2
    style Quote fill:#7c3aed22,stroke:#7c3aed
    style Sales fill:#ea580c22,stroke:#ea580c
    style Supply fill:#dc262622,stroke:#dc2626
    style DB fill:#37415122,stroke:#374151