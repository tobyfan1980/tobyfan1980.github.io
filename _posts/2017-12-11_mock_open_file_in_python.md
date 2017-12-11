---
layout: post
title:  "Mock File Open in Python Unit test"
date:   2017-12-11 11:00:00 -0800
categories:    Programming
tags:    Python
---

A program usually needs to read from a file. But in unit test, we want to mock this with predefined content. For example, the code needs to
read a config file like code below. 

```
import yaml
def load_config():
    with open(constants.CONF_FILE_PATH) as config_file:
        config = yaml.load(config_file.read())

    return config
```

However in test we can define a configuration and convert it to a string. 

```
my_config = {
    'ip': '192.168.0.111',
    'port': 23333
}

my_config_string = yaml.dump(my_config)
```

Then, we define a function to mock the file reading ONLY for the config file name. (This means we don't mock other file reading)

```
def test_config_file_open(filename):
    if filename == constants.CONF_FILE_PATH:
        contents = my_config_string
    else:
        raise IOError("file %s not found" % filename)

    file_object = mock.mock_open(read_data=contents).return_value
    return file_object
```

Finally, we can mock the file reading with out test function.

```
with mock.patch('__builtin__.open', new=test_config_file_open):
    configuration = load_config()
```
