## TimeSeriesAnalysis: Your first simulation


---

### Example 


Defining the models

```csharp
            // define two unit-model and a "PlantSimulator" where the two are connected
            UnitParameters modelParameters = new UnitParameters
            {
                TimeConstant_s = 10,
                LinearGains = new double[] { 1,2 },
                TimeDelay_s = 0,
                Bias = 5
            };
            UnitModel processModel 
                = new UnitModel(modelParameters, "SubProcess1");
            var pidParameters = new PidParameters()
            {
                Kp = 0.5,
                Ti_s = 20
            };
            var pidModel = new PidModel(pidParameters, "PID1");
            var sim = new PlantSimulator (
                new List<ISimulatableModel> { pidModel, processModel  });
            sim.ConnectModels(processModel,pidModel);
            sim.ConnectModels(pidModel,processModel,(int)INDEX.FIRST);
```

Create a synthetic set of input data time-series, which will be simulated over
```csharp
            double timeBase_s = 1;
            int N = 500;
            var inputData = new TimeSeriesDataSet();
            inputData.Add(sim.AddExternalSignal(processModel,SignalType.Disturbance_D),
                TimeSeriesCreator.Step(N/4,N,0,1));
            inputData.Add(sim.AddExternalSignal(pidModel,SignalType.Setpoint_Yset),
                TimeSeriesCreator.Constant(50,N));
            inputData.Add(sim.AddExternalSignal(processModel,SignalType.External_U, (int)INDEX.SECOND),
                TimeSeriesCreator.Step(N/2,N,0,1));
            inputData.CreateTimestamps(timeBase_s);
```

Simulating the PlantSimulator over the dataset:
```csharp
           var isOk = sim.Simulate(inputData,out var simData);
```

Plotting the result:
```csharp
     Plot.FromList(new List<double[]> {
     	simData.GetValues(processModel.GetID(),SignalType.Output_Y),
        simData.GetValues(processModel.GetID(),SignalType.Disturbance_D),
        inputData.GetValues(processModel.GetID(),SignalType.External_U,(int)INDEX.SECOND),
        simData.GetValues(pidModel.GetID(),SignalType.PID_U)
        },
        new List<string> { "y1=y_sim", "y2=disturbance", "y2=u_external","y3=u_pid"  },
        timeBase_s, "ex6_results");
```

which give the following plot:

[Contribution guidelines for this project](figs/gettingStarted_fig1.png)
