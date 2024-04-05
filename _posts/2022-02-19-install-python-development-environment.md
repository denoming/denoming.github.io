
# Installing

## Install anaconda

To install anaconda you should download installation package and run it:
```shell
$ cd ~/Downloads
$ wget https://repo.anaconda.com/archive/Anaconda3-2022.05-Linux-x86_64.sh
$ chmod +x Anaconda3-2022.05-Linux-x86_64.sh
$ ./Anaconda3-2022.05-Linux-x86_64.sh
...
$ 
```

To disable auto initializing of anaconda (optional):
```shell
$ conda config --set auto_activate_base false
$ conda info -e
...
base                  *  /home/denys/.local/opt/anaconda3
```

To update anaconda:
```shell
$ conda update --all
$ conda update conda
---
```

### Install nodejs

```shell
$ wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
$ source ~/.bashrc
$ nvm install v16.13.2
```

### Install jupyter

```shell
$ conda activate
(base) $ conda install -c conda-forge ipympl
(base) $ conda install -c conda-forge jupyterlab
(base) $ jupyter labextension install @jupyter-widgets/jupyterlab-manager
(base) $ jupyter labextension install jupyter-matplotlib
```

### Install jupyter and tensorflow

```shell
$ conda create -n tf tensorflow
(tf) $ conda install -c conda-forge ipympl
(tf) $ conda install -c conda-forge jupyterlab
(tf) $ jupyter labextension install @jupyter-widgets/jupyterlab-manager
(tf) $ jupyter labextension install jupyter-matplotlib
```

## Run file in IPython

```shell
$ conda activate
$ ipython <path-to-file>.py <args>...

# Run Jupyter Notebook
$ conda activate
$ jupyter lab
```

# Configuring

## Configure virtual environment

```bash
$ python3 -m pip install --user --upgrade pip
$ python3 -m pip install --user virtualenv
$ python3 -m pip --version

# Create 
$ python3 -m venv .venv
# Activate
$ source .venv/bin/activate
# Install dependencies from requirements file
$ pip3 install -r requirements.txt
# Deactivate
$ deactivate
```