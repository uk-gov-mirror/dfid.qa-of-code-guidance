# Configuration

Configuration describes how your code runs when you execute it.

In analysis, we may want to run our analysis code using different inputs or parameters. And we likely want others analysts to be able to run our code on different machines, for example, to reproduce our results. This section describes how we can define analysis configuration that is easy to update and can remain separate from the logic in our analysis.

## Basic configuration

Configuration for your analysis code should include high level parameters (settings) that can be used to easily adjust how your analysis runs. This might include paths to input and output files, database connection settings and model parameters that are likely to be adjusted between runs.

In early development of our analysis, lets imagine that we have a script that looks something like this:

```python
data = read_csv("C:/a/very/specific/path/to/input_data.csv") 

variables_test, variables_train, outcome_test, outcome_train = train_test_split(data["a", "b", "c"], data["outcome"], test_size=0.3, random_seed=42)

model = Model()
model.fit(variables_train, outcome_train)

# prediction = model.predict(variables_test, constant_a=4.5, max_v=100)
prediction = model.predict(variables_test, constant_a=7, max_v=1000)

prediction.to_csv("outputs/predictions.csv")
```

Here we're reading in some data and splitting it into subsets for training and testing a model. We use one subset of variables and outcomes to train our model and then use the other variables subset to test the model. Finally, we write the model's predictions to a `.csv` file.

The file paths that are used to read and write data in our script are particular to our working environment. These files and paths may not exist on another analysts machine. As such, other analysts would need to read through the script and replace these paths in order to run our code. As we'll demonstrate below, collecting flexible parts of our code together makes it easier for others to update them.

When splitting our data and using our model to make predictions, we've provided some parameters to the functions that we have used to perform these tasks. Eventually, we might reuse some of these parameters elsewhere in our script (e.g. the random seed) and we are likely to adjust these parameters between runs of our analysis. To make it easier to adjust these consistently throughout our script, we should store them in variables. We should also store these variables with any other parameters and options, so that it's easy to identify where they should be adjusted.

Note that in this example we've tried our model prediction twice, with different parameters. We've used comments to switch between which of these lines of code runs. This practice is common, especially when we want to make a number of changes when developing how our analysis should run. However, commenting sections of code in this way makes it difficult for others to understand our code and reproduce our results. Another analyst would not be sure which set of parameters was used to produce a given set of predictions, so we should avoid this form of ambiguity. Below, we'll look at some better alternatives to storing and switching our analysis parameters.


```python
# Configuration
input_path = "C:/a/very/specific/path/to/input_data.csv"
output_path = "outputs/predictions.csv"

test_split_proportion = 0.3
random_seed = 42

prediction_parameters = {
    "constant_a": 7,
    "max_v": 1000
}


# Analysis
data = read_csv(input_path)

variables_test, variables_train, outcome_test, outcome_train = train_test_split(data["a", "b", "c"], data["outcome"], test_size=test_split_proportion, random_seed=random_seed)

model = Model()
model.fit(variables_train, outcome_train)

prediction = model.predict(variables_test, constant_a=prediction_parameters["constant_a"], max_v=prediction_parameters["max_v"])

prediction.to_csv(output_path)
```

Separating configuration from the rest of our code makes it easy to adjust these parameters and apply them consistently throughout the analysis script. We're able to use basic objects (like lists and dictionaries) to group related parameters. We then reference these objects in the analysis section of our script.

Our configuration could be extended to include other parameters, including which variables we're selecting to train our model. However, it is important that we keep the configuration simple and easy to maintain. Before moving aspects of code to the configuration it's good to consider whether it improves your workflow. If it is something that is dependent on the computer that your using (e.g. file paths) or is likely to change between runs of your analysis, then it's a good candidate for including in your configuration.

## Configuration files

We can use independent configuration files to take our previous example one step further. We simply take our collection of variables, containing parameters and options for our analysis, and move them to a separate file. As we'll describe in the following subsections, these files can be written in the same language as your code or other simple languages.

Storing our analysis configuration in a separate file to the analysis code is a useful separation. It means that we can version control our code based solely on changes to the overall logic - when we fix bugs or add new features. We can then keep a separate record of which configuration files were used with our code to generate specific results. We can easily switch between multiple configurations, by providing our analysis code with different configuration files.

You may not want to version control your configuration file, for example if it includes file paths that are specific to your machine or references to sensitive data. In this case, you should include a sample or example configuration file, so that others can use this as a template to configure the analysis for their own environment. It is key that this template is kept up to date, so that it is compatible with your code.

### Code configuration files

To use another code script as our configuration file, we can copy our parameter variables directly from our scripts. Because these variables are defined in the programming language that our analysis uses, it's easy to access them in our analysis script. In Python, variables from these config files can be imported into your analysis script. In R, your script might `source()` the config file to read the variables into the R environment.

### Dedicated configuration files

Many other file formats can be used to store configuration parameters. You may have come across data-serialisation languages (including YAML, TOML, JSON and XML), which can be used independent of your programming language.

If we were to represent our example configuration from above in YAML, this would look like:

```
input_path: "C:/a/very/specific/path/to/input_data.csv"
output_path: "outputs/predictions.csv"

test_split_proportion: 0.3
random_seed: 42

prediction_parameters:
    constant_a: 7
    max_v: 1000
```

Configuration files that are written in other languages may need to be read using relevant libraries. The YAML example above could be read into our analysis as follows:

```
import yaml

with open(r"./my_config.yaml") as file:
    config = yaml.load(file)

config

# Analysis
data = read_csv(config$input_path)
...
```

Configuration file formats like YAML and TOML are compact and human-readable. This makes them easy to interpret and update, even without knowledge of the underlying code used in the analysis. Reading these files in produces a single object containing all of the `key:value` pairs defined in our configuration file. In our analysis, we can then select our configuration parameters using their keys.
