This is an implementation of genetic algorithm evolution to develop strategies for digital coin trading bot <a href="https://github.com/askmike/gekko">Gekko</a>. 

It generates random configs & evolve them by backtesting on a Gekko session via the REST API of gekko's user interface. <br>
Genetic algorithm and quick bayesian optimization are evolution choices.


### Setup

Install gekko, then clone this repo and install dependencies:
```
$ git clone https://git.com/Gab0/gekkoJaponicus.git
$ cd gekko_japonicus
$ sudo pip install -r requirements.txt
```

### Usage

```
Open two terminals;

T.1 -> run Gekko on ui mode, or just its webserver:
$node gekko.js --ui
        or
$node web/server.js

T.2 -> $cd [japonicus dir]
       $python japonicus.py [-g|-b] [-c] [-k] [--repeat <X>] [ [-i|-r|--strat <Strategy>] [-w]

    [main optimization options]
    -g for genetic algorithm;
    -b for bayesian optimization;
    
    [optional GA methods]
    -c to use an alternative ~experimental genetic algorithm;

    [strategy selector] 
    -r run with random strategy
    --strat choose one strat to run;
    -i Genetic algorithm create strategies w/ combined indicators on the fly and run 'em.
    
    [repeat runs]    
    --repeat to run genetic algorithm X times; then just check evolution.log;

    [evolution visualization options]
    -w launches a neat dash/flask web server @ your local machine, which can be accessed via  webbrowser.
           Address is shown on the first line of console output. (likely http://localhost:5000)

    -k launches a child gekko instance, so no need for the first terminal;
```

This is written on python because of the nice DEAP module for genetic algorithm, and was worth it. DEAP is required, available on PIP.

All settings are set at Settings.py;

If your Gekko UI http port is not :3000, adjust it;

Backtesting is parallel, running a pool of five workers, adjustable.


### Gekko Strategies

To run japonicus you must select a strat via `--strat <strategy>`, where `<strategy>` corresponds to any working strategy present on gekko `strategies` folder.
The settings ranges for given strategy must be present as a key on `configStrategies.py`. A random strategy may be also selected with `-r` arg.

Those strategies present in a fresh clone of gekko are no good, ditch them. 
With genetic algorithm optimization, those strats can get to a level of profit that is above the market base price change 
for given period. Even then, that power will be only shown at the dataset it was optimized for, so the settings
will fail on different candlesticks which include real time trading.

A good thing to do is to create a strategy that combines two indicators, IE you buy when both results are above buying thresolds,
you sell when both are below selling thresholds. So dual or even triple indicator strategies are a good path to go.
Strategy must go beyond simple logic of indicator results on thresholds to have
meaningfun results.

Custom strategies should be added to configStrat.py, by strategy name.

Known good gekko strategies to run with this:
 - PPO+TSI strat


### Remote Amazon EC2 Cluster

Japonicus can send backtest requests across the internet to several machines running Gekko.
This method can greatly cut EPOCH times.
Intended to use with Ansible-playbook + Amazon EC2 AWS machines. <br>
At japonicus side, you should provide Ansible's `hosts` inventory file path (on settings.generations), containing
the IPs of running machines (a simple list). <br>
Those machines should be already fully configured, running Gekko, and loaded with the same candlestick history data
files you have on local Gekko;<br>
YML playbook file to get gekko running on Amazon AMI Linux is at root folder (set the path to your local gekko history @ line 57);

```
steps to put your GA online on clusters;

1- Create how many EC2 instances you want - spot requests are cheaper;

2- Make sure port 3000 TCP is open on them. This is configurable on security groups.
    Theres a nice tutorial at https://www.google.com/search?q=amazon+aws+security+groups XD
    
3- Find a way to install node and gekko on them. Copy your local gekko/history folder to them;
(there is a ansible playbook on the repo that can help you cover this steps)

4- Make a list of your remote machine addrs and inform the path to it at Settings.py->Global.GekkoURLs. A hosts file used by ansible can be used as it is.

5- Run japonicus.py in any GA mode;

```

### Docker Compose

You can include japonicus to gekko image by adding the following snippet to gekkos docker-compose file

```yml
  japonicus:
    build: relative-path-from-gekko-to-the-japonicus-repo/
    volumes:
      - ./volumes/gekko-japonicus:/usr/src/app/output
    links:
      - gekko
    ports:
      - 5000:5000
```

and by setting the `GekkoURLs` config to the container gekko name
```python
# Settings.py
'GekkoURLs': ['http://gekko:3000']
```

japonicus will choose a random item of the list to fetch candles and scansets.

### Future

Genetic Algorithms are a good way to fetch a good set of settings to run a strategy
on gekko. But the real gamechanger is the strategy itself. The ideal evolution method
would be a Genetic Programming that modifies the strat logic itself. This somewhat
corresponds to -i option of japonicus, still barebones.
