# Measuring the impact of nonpharmaceutical interventions on the SARS-CoV-2 pandemic at a city level: An agent-based computational modelling study of the City of Natal

This model is based on agents (citizens) and simulates the interaction between agents on a daily basis, the transmission of the disease (Covid19) between the agents and the disease progression in the agent. The differential of this model will be the implementation of different modes of interaction between agents, simulating the different means that can be used in eventual non-pharmaceutical strategies to combat the epidemic.

The algorithm consists of a cycle for one day. The cycle contains two parts: computing the interactions of each agent with other agents considering all modes of interaction; and updating the status of each agent.

# Software requirements:
- Python 3.6+
- Additional modules for simulation (install with `pip install jsonmerge numba`):
    - jsonmerge: To simplify merging input parameter files
    - numba: For code optimisation
- For output analysis `pip install matplotlib`:
    - matplotlib: graph visualisation
- For testing `pip install pytest coverage`:
    - pytest: For system and unit tests
    - coverage: Generate code coverage reports 
- For optimization:`pip install pyswarms` and `pip install deap`:
    - yswarms: For PSO
    - deap: For evolutionary computation framework
# Project Structure

- `covidsimulation` is the root of project code (the interface will be described below in inputs/outputs)
- `covidsimulation/data/`
    - All the neighborhoods information from Natal city.
    - Utility files to convert the raw neighborhood data into city parameters used in the algorithm.
    - Example input parameters to the simulation
    - Example inputs to the optimisation algorithm
- `covidsimulation/analysis/`
    - `graph.py` plot the simulation results.
    - `optimise.py` learn simulation parameters which fit some observed health outcomes. 
- `covidsimulation/simulation/`
    - Main package, implements the agents, complex network and data collection.
- `covidsimulation/common/`
    - Shared code between the packages.
- `covidsimulation/tests/`
    - Regression/unit tests check for breaking changes and should be run successfully before merging code.
    - Profiling analyses how long the algorithm takes to run and where time is spent (uses `snakeviz` for visualisation).

## Simulation

### Inputs/Outputs

call `covidsimulation` with the following arguments:
- `-o <file_path>`: specify where to output the result of the simulation
- `--param-files <file1> <file2> ... <filen>`: specify the filepaths to parameters files containing epidemy, city, and mobility information.
- `--o-network <file_path>`: specify where to output the network used in the simulation (optional).
- `--i-network <file_path>`: specify an input file containing a complex network description (optional).
- `--multi-run <number>`: specify a number of simulations to be run (optional, default=1).
- `--full-agents`: if specified along with `--o-netwok` then the output network will contain the full agents state allowing you to "resume" the simulation from the state of the finished simulation. 

Example call from project root: `python -m covidsimulation --param-files ./covidsimulation/data/params/tests/params_example.json`

Once the simulation is complete a data file will be generated. running `graph.py` in `output_analysis/` will visualise different aspects of the generated health outcomes data. example call `python -m covidsimulation.analysis.graph ./covidsimulation/analysis/sim_results/output.json`

### Inputs in detail 

Parameter files (`--param-files`) are json format files which contain variables used by the simulation, the format for the parameters is fixed, examples are provided in `covidsimulation/data/params`. 

By default a complex network will be generated based on the above parameters. Alternatively, you can supply a pregenerated network (using `--i-network`) or a simulation generated network from a previous run (from the `--o-network` instruction). Optionally, you can specify the `--full-agents` flag which will cause the network information to be output with additional agent state information, doing this allows you to "resume" the simulation from the network final state.

To allow for more efficient data collection the `--multi-run` parameter allows you to request multiple independent simulations to be run (will make use of all available CPU cores).

## Optimisation

call `covidsimulation.analysis.optimise` with the following arguments:
- `--param-files <file1> <file2> ... <filen>`: same as simulation, specify the filepaths to parameters files containing epidemy, city, and mobility information.
- `--free-params <file_path>`: the file containing the parameters to be tuned. the format of the file and the parameters that can be used for optimisation are **fixed** (the number of "event" is variable).
- `--mode {EA, PSO}`: choose the optimisation strategy (optional, default=EA).
- `<file_path>`: last argument is the data to be fitted.

example call:
`python -m covidsimulation.analysis.optimise --param-files ./covidsimulation/data/params/tests/params_ScabiniEtAl.json ./covidsimulation/data/real_outcomes/days.json --free-params ./covidsimulation/data/real_outcomes/free_params.json ./covidsimulation/data/real_outcomes/real_data.json`

output:
- Visualise the best individual (output.png)
- the best parameters found during the optimisation session (output_free_params.json)

### Inputs in detail

All parameters are encoded in the json format but each contain distinct information, this section explains what each of the arguments to the optimiser means.

Simulation parameter files (i.e. `--param-files`) are the same as in the case of running the simulation. They should be considered the "static" parameters which don't need to be tuned. 

Free parameters are contained in a single file (passed with `--free-params`) and have a separate format to the simulation parameters. These are the "dynamic" parameters which will be tuned by the optimisation process. Each free parameter corresponds to a simulation parameter and will be translated into the simulation format, more info in next section.

The last argument to the optimisation command line tool is a single file which contains both the target data (number of confirmed cases/deaths per day) and options to the optimisation process. The latter is for controlling the number of iterations, simulations, population size and have a large impact on the time taken to run and the likelihood of identifying a good solution.

Examples of all of the above can be found in `covidsimulation/data/real_outcomes`. `free_params.json` contains the currently learned parameters, `free_params_schema.json` shows an example of the format for these parameters. `real_data.json` is an example of the last argument to the optimisation command. Finally, there is a utility python script `convert.py` which will generate these files based on the csv data, editing this file may be faster than manually rewritting the json.

### Adding a new parameter

Check the `covidsimulation/data/real_outcomes/free_params_schema.json` for the format of free parameters.

Any positive numerical parameter used in the simulation can be designated a free parameter. the `path` elements must match the structure expected in the simulation as this is used to convert from this representation to the simulation representation. The optimisation algorithm works on a vector of floating point numbers between 0.0 and 1.0, `scale` is multiplied to the value in the solution vector, this allows integer parameters to be derrived.

In the case of using the `free_params.json` file mentioned above, adding a parameter to be optimised is done simply by adding an entry into this file which corresponds to some parameter in the simulation. If the parameter you are tuning is already in the "static" parameters, then the learned parameter will overwrite the "static" version. 

Note: Support for negative parameters and array parameters is not currently implemented. Hyperparameter tuning is also not implemented.

# Contributing to the project

Work on branches, generate a pull request to master when you want to push changes, make sure any tests pass and have _someone_ review the changes before merging. 

## Testing

run `pytest` from the root of the directory to run the tests.

To add a test make sure you follow the pytest convention:
- tests must be in files named `test_<name>.py`
- tests must be in functions named `test_<name>.py`

you don't need to call the function just have it exist.

To generate a code coverage report instead run the command `coverage run -m pytest` followed by `coverage html` to get an easy to read report (open `index.html` in the generated `htmlcov` directory).

