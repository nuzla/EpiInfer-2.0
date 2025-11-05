```mermaid
graph TB
    Start([Start: Epidemic Forecasting]) --> InputCheck{What Data<br/>Available?}
    
    InputCheck -->|Full Contact Tracing| ContactPath[Contact Tracing Path]
    InputCheck -->|Only Aggregate Stats| DistPath[Meeting Distribution Path]
    
    %% Contact Tracing Path
    ContactPath --> Input1[/"📊 Inputs:<br/>- Contact pairs per day<br/>- Initial infected/exposed<br/>- Population size<br/>- inc, duration_infected, duration_recovered"/]
    
    Input1 --> Calibrate1{Known<br/>p1, p2?}
    Calibrate1 -->|No| Alg5a["🔧 Algorithm 5<br/>ContinuousCalibrate<br/>-----------------<br/>Find optimal p1, p2<br/>using observed data"]
    Calibrate1 -->|Yes| UseParams1[Use provided p1, p2]
    Alg5a --> GotParams1[Parameters Ready]
    UseParams1 --> GotParams1
    
    GotParams1 --> Alg1["🎯 Algorithm 1<br/>CONTACTINFER-CORE<br/>-----------------<br/>Main simulation loop<br/>Day-by-day prediction"]
    
    Alg1 --> DayLoop1{For each day d}
    
    DayLoop1 --> Alg2["📍 Algorithm 2<br/>PREDICT<br/>-----------------<br/>1. Calculate exposure prob<br/>2. Expected new exposures<br/>3. Select top N by risk"]
    
    Alg2 --> Alg3["🔍 Algorithm 3<br/>FINDPROB<br/>-----------------<br/>For each susceptible:<br/>P(exposure) = 1-(1-p1)^contacts"]
    
    Alg3 --> NewExp[New Exposed Set]
    NewExp --> CheckInf{Day d+1-inc<br/>exists?}
    
    CheckInf -->|Yes| ApplyP2["Apply p2:<br/>Each exposed → infected<br/>with probability p2"]
    CheckInf -->|No| Skip1[No new infections yet]
    
    ApplyP2 --> NewInf1[New Infected Set]
    Skip1 --> UpdateStates1
    NewInf1 --> UpdateStates1["Update States:<br/>- Active infections<br/>- Recoveries<br/>- Susceptible pool"]
    
    UpdateStates1 --> MoreDays1{More days<br/>to predict?}
    MoreDays1 -->|Yes| DayLoop1
    MoreDays1 -->|No| Results1
    
    %% Meeting Distribution Path
    DistPath --> Input2[/"📊 Inputs:<br/>- Daily new infections (observed)<br/>- Mean contacts per day<br/>- Asymptomatic & Recovered counts<br/>- inc, p1*, p2* (*=unknown)"/]
    
    Input2 --> Alg5b["🔧 Algorithm 5<br/>ContinuousCalibrate<br/>-----------------<br/>Grid search p2: 0.1→1.0<br/>Binary search p1: 0→1<br/>Minimize RMSE on training"]
    
    Alg5b --> TrainLoop{For each<br/>training day}
    
    TrainLoop --> TestParams["Test p1, p2 pair<br/>using Algorithm 4"]
    TestParams --> CalcRMSE["Calculate RMSE:<br/>predicted vs observed"]
    CalcRMSE --> BestParams{Better than<br/>best so far?}
    BestParams -->|Yes| UpdateBest[Update best p1, p2]
    BestParams -->|No| KeepBest[Keep current best]
    UpdateBest --> MoreTests{More<br/>parameters?}
    KeepBest --> MoreTests
    MoreTests -->|Yes| TrainLoop
    MoreTests -->|No| GotParams2[Optimal p1, p2 Found]
    
    GotParams2 --> Alg4["⚙️ Algorithm 4<br/>EPIINFER-CORE<br/>-----------------<br/>Meeting distribution approach<br/>Uses Equations 1-5"]
    
    Alg4 --> Eq1["📐 Equation 1:<br/>palreadyexp = Σ(NewInf)/p2×asymp<br/>(fraction already exposed)"]
    
    Eq1 --> Eq2["📐 Equation 2:<br/>pnewexp = 1-(1-p1)^(contacts×palreadyexp)<br/>(individual exposure probability)"]
    
    Eq2 --> Eq3["📐 Equation 3:<br/>asymptomaticNotEx = asymp - exposed - recovered<br/>(available susceptible population)"]
    
    Eq3 --> Eq4["📐 Equation 4:<br/>NewExposed = asymptomaticNotEx × pnewexp<br/>(expected new exposures)"]
    
    Eq4 --> Eq5["📐 Equation 5:<br/>NewInf(d) = NewExposed(d-inc) × p2<br/>(predicted new infections)"]
    
    Eq5 --> MoreDays2{More days<br/>to predict?}
    MoreDays2 -->|Yes| Alg4
    MoreDays2 -->|No| Results2
    
    %% Results
    Results1 --> Output[/"📈 Outputs:<br/>- Daily predictions (1-20 days)<br/>- Total infected forecast<br/>- RMSE & accuracy metrics<br/>- Forecast tables"/]
    Results2 --> Output
    
    Output --> Evaluate{Evaluate<br/>Performance}
    
    Evaluate --> Compare["Compare Methods:<br/>✓ EpiInfer-CONTACT<br/>✓ ARIMA<br/>✓ LSTM<br/>✓ Differential Equations"]
    
    Compare --> Insight["📌 Paper Finding:<br/>ARIMA best for ≤5 days<br/>EpiInfer-CONTACT best for 6+ days<br/>Hybrid: up to 4× better RMSE"]
    
    Insight --> Update{New data<br/>available?}
    
    Update -->|Yes| Recalibrate["Re-run Algorithm 5<br/>with updated data<br/>(continuous calibration)"]
    Recalibrate --> Alg1
    Recalibrate --> Alg4
    Update -->|No| End([End: Forecasts Ready])
    
    %% Styling
    classDef inputStyle fill:#e1f5ff,stroke:#0288d1,stroke-width:2px
    classDef algoStyle fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef eqStyle fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef outputStyle fill:#e8f5e9,stroke:#388e3c,stroke-width:2px
    classDef decisionStyle fill:#fff9c4,stroke:#f9a825,stroke-width:2px
    
    class Input1,Input2 inputStyle
    class Alg1,Alg2,Alg3,Alg4,Alg5a,Alg5b algoStyle
    class Eq1,Eq2,Eq3,Eq4,Eq5 eqStyle
    class Output,Results1,Results2,Insight outputStyle
    class InputCheck,Calibrate1,CheckInf,MoreDays1,MoreDays2,BestParams,MoreTests,Evaluate,Update decisionStyle
```
