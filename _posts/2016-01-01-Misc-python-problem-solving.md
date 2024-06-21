---
layout: post
title: Misc python problem solving
---
<!--<img src="/images/fulls/01.jpg" class="fit image">-->
This post will have various small examples of the use of different python tools and frameworks that I was not able to fit into the bigger examples, but I find worthy of showcasing.

## Load environment variables

It is important to handle enviroment variables delicately as we should not have usernames and passwords floating around in the github repos. A simple way of doing this is to have all creds for connecting to databases and APIs in a .env file that is added to the .gitignore file and thereby not pushed to the remote repo. There are even more secure ways of doing handling this.

In side the python files that are used in production you could then add this code to load the .env file contents if you run the file locally. However, this would fail if the python file was run elsewhere.

```python
try:
    from dotenv import load_dotenv, find_dotenv
    load_dotenv(find_dotenv(), override=True)
except:
    pass
```

Obviously we would need to be able to run the python files in the cloud for some scheduled tasks. This could for example be as a lambda function on AWS or as a containerised version of the repo. The environment variables would then be set up securely in this environment so the code would run as intended.

## Logging

When running code in production it is important to have good logging set up, so you can more easily locate problems with the scheduled code as it runs in the cloud. And also to give good info on the status of the run.

To set it up in the python files we need to import logging and define the logger.

```python
import logging

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

formatter = logging.Formatter('%(asctime)s:%(levelname)s:%(name)s:%(message)s')

file_handler = logging.FileHandler('script.log')
# file_handler.setLevel(logging.ERROR) # If you want to set different level for the file it self
file_handler.setFormatter(formatter)

stream_handler = logging.StreamHandler(sys.stdout)
stream_handler.setFormatter(formatter)

logger.addHandler(file_handler)
logger.addHandler(stream_handler)
```

We can now trigger the logger with these methods:
debug(), info(), warning(), error() and critical()

For example:

```python
logger.info('Initialising {} dataset preparation...'.format(dataset_name))
```

Usually you will print these to a log file as well to be able to always view the history of previous runs and be able to determine a problem this way.