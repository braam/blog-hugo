---
title: "Cronjobs Manager Cronicle"
date: 2022-07-05T16:50:52+02:00
draft: false
tags:
- Linux
---

Running cronjobs is easy, but a nice interface to manage them will make everything even better. Meet [cronicle](http://cronicle.net/) it's an opensource project hosted on [github](https://github.com/jhuckaby/Cronicle)!

Cronicle is a multi-server task scheduler and runner, with a web based front-end UI. It handles both scheduled, repeating and on-demand jobs, targeting any number of slave servers, with real-time stats and live log viewer. 
You can create user groups who can view and execute only their scripts.

![Cronicle](/posts_images/cronicle_dashboard.png)

Full installation instructions can be found at [https://github.com/jhuckaby/Cronicle#installation](https://github.com/jhuckaby/Cronicle#installation)
```bash
curl -s https://raw.githubusercontent.com/jhuckaby/Cronicle/master/bin/install.js | node
```

&nbsp;
### Running Python Scripts With Cronicle
You can execute bash scripts or any other commands directly from the shell script launcher. To run python scripts, you can manage it like this:

![Cronicle Python](/posts_images/cronicle_python_script.png)

&nbsp;
### Passing Errors to Cronicle
When you launch a shell script or python script, you can pass the error when your script is failing.
It's enough to exit your script with exit 1 (because 0 is always succesful).
For python we can use it like this:
```python
sys.exit('ERROR >> TOKEN << check log file for the details.')
```

Cronicle can use this error message in your notification on failure.

