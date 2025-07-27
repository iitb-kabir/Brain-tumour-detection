Project Title
This repository contains the necessary files to set up a Python environment for your project, likely related to neuroimaging given the environment name in the instructions.

Setup Instructions
Follow these steps to set up your development environment using Conda:

1. Save the Environment File
First, save the provided environment configuration as environment.yml in the root directory of your project.

2. Create the Conda Environment
Open your terminal or Anaconda Prompt and navigate to the directory where you saved environment.yml. Then, run the following command to create the Conda environment:

conda env create -f environment.yml

This command will read the environment.yml file and install all the specified dependencies.

3. Activate the Environment
Once the environment is created, activate it using the following command:

conda activate neuroimaging-env

You should now be in the neuroimaging-env environment, ready to run your project's code.

Additional Notes
If your project requires GPU support, ensure your Conda installation and environment are configured accordingly.

If you need additional packages not listed in environment.yml (e.g., tifffile), you can install them after activating the environment using conda install <package-name> or pip install <package-name>.
