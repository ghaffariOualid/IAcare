# IAcare
graph TD
    %% Define styles for clarity
    classDef security fill:#a0ffa0,stroke:#008000,stroke-width:2px;
    classDef client_op fill:#ffb366,stroke:#cc5500,stroke-width:2px;
    classDef server_op fill:#66aaff,stroke:#0055cc,stroke-width:2px;
    classDef comm fill:#ffffe0,stroke:#aaaa00,stroke-width:1px;
    classDef data_sensitive fill:#ffcccc,stroke:#cc0000,stroke-width:2px;

    subgraph I[Architecture FedShield-LLM Sophistiquée (FHE + Pruning)]
        direction LR
        
        subgraph C[Client I (Entreprise Conteneurisée)]
            C_DATA[(Documents Sensibles BERT Input)]
            C_LORA[1. Entraînement Local BERT/LoRA (Analyse/Classif.)]
            C_PRUNE[2. Élagage (Pruning) Agresif des w_i]
            C_ENCRYPT[3. Chiffrement Homomorphe (FHE) : Enc(w_i)]
            C_METRIC[Envoi Métriques (Loss) à Kafka]
            
            C_DATA --> C_LORA
            C_LORA --> C_PRUNE
            C_PRUNE --> C_ENCRYPT
            C_LORA --> C_METRIC
            
            class C_PRUNE,C_ENCRYPT security
            class C_LORA client_op
            class C_DATA data_sensitive
        end
        
        subgraph S[Serveur Central (Orchestrateur)]
            S_RECV[Réception Enc(w_i) via Volume Partagé]
            S_AGGR_FHE[4. Agrégation Sécurisée (FHE) : Calc sur Enc(w_i)]
            S_DECRYPT[5. Déchiffrement du Résultat : w^{t+1}]
            S_SEND[Distribution w^{t+1} (Poids LoRA Globaux)]
            
            S_RECV --> S_AGGR_FHE
            S_AGGR_FHE -- Résultat Agrégé Chiffré --> S_DECRYPT
            S_DECRYPT --> S_SEND
            
            class S_AGGR_FHE,S_DECRYPT security
            class S_RECV,S_SEND server_op
        end
        
        subgraph MONITORING[Monitoring Distribué]
            KAFKA(Broker Kafka)
            DASH[Dashboard Streamlit (Visualisation Loss)]
            KAFKA --> DASH
            class KAFKA,DASH comm
        end
        
        % Communications
        C_ENCRYPT -- Flux de Poids : Enc(w_i) --> S_RECV
        S_SEND -- Flux de Poids : w^{t+1} --> C_LORA
        C_METRIC -- Flux de Données : Loss --> KAFKA
    end
    
    subgraph DOCKER[Environnement Docker Compose]
        DOCKER_ISOLATION((Isolation de N Clients))
        DOCKER_ISOLATION --> C
    end
    
    style I fill:#f0f8ff,stroke:#007bff
