flowchart LR
    A["Source Service"] --> B["Message Broker 1"]

    subgraph B["Message Broker 1"]
        C["{ txlog_id: xxx }"]
    end

    B --> D["Prepare Function"]

    subgraph D["Prepare Function"]
        D0["Get hisdata by txlog_id"]
        D1["Get pipeline config from DB by $pre_func_name"]
        D2["Execute prepare function"]
        D3["Insert into $pre_func table"]
        D0 --> D1 --> D2 --> D3["Insert into $pre_func table with $txlog_id"]
    end

    D --> E["Message Broker 2"]

    subgraph E["Message Broker 2"]
        F["{
txlog_id: xxx,
pipeline_config: {
id: xxx,
pre_func_name: xxx,
agg_config: {},
kernal_config: []
}
}"]
    end

    E --> G["Aggregate Function"]

    subgraph G["Aggregate Function"]
        G1["Get prepared result from $pre_func table filter by txlog_id"]
        G2["Parse agg_config from event"]
        G3["Execute aggregate function"]
        G4["Insert aggregate result with key: $pipeline_id:$agg_col1_val:$agg_col2_val:..."]
        G1 --> G2 --> G3 --> G4
    end

    G --> H["Message Broker 3"]

    subgraph H["Message Broker 3"]
        I["{<br>txlog_id: xxx,<br>pipeline_config: {<br>  id: xxx,<br>  kernal_config: []<br>},<br>aggregate_result: []<br>}"]
    end

    H --> K["Kernel Function"]

    subgraph K["Kernel Function"]
        K1["Parse kernal_config & aggregate_result from event"]
        K2["Execute kernal function"]
        K3["Execute action flow"]
        K1 --> K2 --> K3
    end

    %% 🎨 顏色樣式設定
    style D fill:#ffe6f2,stroke:#ff99cc,stroke-width:2px
    style D0 fill:#ffffff,stroke:#000,stroke-width:1px
    style D1 fill:#ffffff,stroke:#000,stroke-width:1px
    style D2 fill:#ffffff,stroke:#000,stroke-width:1px
    style D3 fill:#ffffff,stroke:#000,stroke-width:1px

    style G fill:#e6f2ff,stroke:#3399ff,stroke-width:2px
    style G1 fill:#ffffff,stroke:#000,stroke-width:1px
    style G2 fill:#ffffff,stroke:#000,stroke-width:1px
    style G3 fill:#ffffff,stroke:#000,stroke-width:1px
    style G4 fill:#ffffff,stroke:#000,stroke-width:1px

    style K fill:#e6ffe6,stroke:#33cc33,stroke-width:2px
    style K1 fill:#ffffff,stroke:#000,stroke-width:1px
    style K2 fill:#ffffff,stroke:#000,stroke-width:1px
    style K3 fill:#ffffff,stroke:#000,stroke-width:1px
