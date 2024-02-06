## TimeSeriesAnalysis: Your first simulation

---

Define a feedback loop consisting of a linear dynamic "unitmodel" with two inputs describing the process and a PID-controller.  

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

A set of external signals is defined:
- to give in a disturbance to the processModel, 
- a setpoint to the PID-controller, and
- a second non-pid controlled input to the processModel.

The below code creates these external signals, adds steps to the disturbance and setpoint, and places them 
in the ``inputData'' dataset:

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

The above code creates the following "plant":
![Unit model layout in plant](https://steinelg.github.io/steinelg/figs/gettingStarted_fig2.png)

which can now be simulated with a single line of code, the results of the simulation are stored in ``simdata''.

```csharp
           var isOk = sim.Simulate(inputData,out var simData);
```

The result of the simulation could now be used for further analysis. 

When debugging, it can be useful to plot the ``inputData'' and ``simData'' to see how the simulation went:

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

which creates the following plot:

![Figure of time-series added by example](https://steinelg.github.io/steinelg/figs/gettingStarted_fig1.png)

The PID-controller output, the manipulated variable, moves both in response to the setpoint change
and to the disturbance, and as the process and PID-controller are both dynamic, two transients are created in the dataset. 
This is a dynamic simulation, and if the models had been fitted to match an actual process, and the ``inputData'' was obtained 
from measurements*, the above model could be termed a *dynamic digital twin*.

(*=Note that the disturbance signal is not usually measured, instead it is possible to estimate this variable.)


