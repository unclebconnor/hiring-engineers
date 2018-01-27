# Brian Connor's excellent adventure through the hiring exercise

## Notes on setup
* I was able to install vagrant and the datadog agent with no trouble INCLUDE LINK TO VAGRANT RESOURCE AND RELEVANT COMMANDS
* I confirmed that the agent was reporting by looking at the host map page INCLUDE LINK OR REFER TO IMAGE
* I also installed an agent in my osx terminal, hoping that I'd get data from two hosts to compare, and because I'm less familiar with the linux vm.  My host map page has changed a few times and at one point I had 3 hexagons, 2 of which represented my macbook.  From this [help article](https://goo.gl/Zm5rY4) It seems that there are cases where a host might send multiple unique names, which Datadog then aliases separately.  The problem seems to have resolved itself (after a few rounds of uninstalling and reinstalling).  
INCLUDE SOME COMMON PATHS AND COMMANDS FOR EACH HOST  
* osx agent path
* osx restart/info
* linux agent path
* linux restart/info
* vi resources/commands
  NECESSARY INSTALLATIONS...Flask, Python, Go, etc

---

## Collecting Metrics:
* Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.  
> I was able to successfully add tags on both the osx and linux hosts.  I had a bit of trouble figuring out how to edit text on Vagrant at first (I used vi) but eventually made it work.  There was a significant lag updating the UI from both hosts, but the VM took much longer.  I did a series of restarts for the agents and logged out/in a few times.  I'm not sure if those were necessary or if I just needed to be patient and I could anticipate a client being similarly confused.  I'm curious what the expected or recommended protocol is.  
  UPDATE: DIRECTIONS FOR UPDATING TAGS.  INCLUDE NOTE THAT THERE IS A LAG AND A REFRESH HAPPENED.
  
  ![Tags Created Successfully](https://github.com/unclebconnor/hiring-engineers/blob/master/images/01_tag-example.png)
  
* Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.
> I don't know why it wasn't obvious at first to rename the "example" file, but once I figured that out the installation was smooth on both hosts, though I'm still running into a sort of annoying issue with permissions on vagrant.  I'm sure I'll figure that out by the time I'm done.  
  UPDATE:  CLARIFY TO RENAME THE EXAMPLE FILE AND WHAT TO CHANGE
  
  ![Postgres Installed Successfully](https://github.com/unclebconnor/hiring-engineers/blob/master/images/02_Postgres-Install.png)
  
* Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.
> The environment(s) are starting to make much more sense.  I've only done some basics on python so had to check on some syntax, but otherwise this was fairly straight-forward.
  UPDATE: INCLUDE CODE SNIPPET AND PYTHON INSTRUCTIONS
  
  ![Agent Check Installed Successfully](https://github.com/unclebconnor/hiring-engineers/blob/master/images/03_agent-check.png)
  
* Change your check's collection interval so that it only submits the metric once every 45 seconds.
> On my linux host, I changed this via the yaml file
  UPDATE: INCLUDE SNIPPET AND SCREENSHOTS OF EACH
  
  ![Collection Interval Updated](https://github.com/unclebconnor/hiring-engineers/blob/master/images/04_mymetric-interval.png)
  
* **Bonus Question** Can you change the collection interval without modifying the Python check file you created?
> For my osx host, I changed this via the settings page for my_metric.  From what I can tell on the line graph, it's still only showing intervals every 20 seconds so I'm wondering if this field requires special statsd syntax.  I wasn't able to find any clear documentation with an answer to that so I'm going to move on for the moment.  Hopefully I'll be able to circle back after doing some more with data visualization.
  
  ![Interval Updated Manually](https://github.com/unclebconnor/hiring-engineers/blob/master/images/05_Challenge-statsd-interval.png)

---

## Visualizing Data:

Utilize the Datadog API to create a Timeboard that contains:

* Your custom metric scoped over your host.
* Any metric from the Integration on your Database with the anomaly function applied.
* Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket
> To complete this task, I actually created a timeboard manually from the UI first to get a good sense of what I was trying to make with the script.  After doing that, I stumbled upon the JSON formatted code for my requests so from that point I just took the template from the API docs and modified it to include the 3 graphs and copy/pasted what I already created.  I used Ruby because I've got a little more experience with it than I do python.  I had to install the dogapi gem, ran the script, and then verified that it was successful on my dashboard list.
  UPDATE:  REWRITE AS DIRECTIONS
  
  Pictured below:
  1. The manually created timeboard
  2. The timeboard created via the API and the script
  
  ![Manual Timeboard](https://github.com/unclebconnor/hiring-engineers/blob/master/images/06a_timeboard-manual.png)
  ![Timeboard from API](https://github.com/unclebconnor/hiring-engineers/blob/master/images/06b_timeboard-from-api.png)
  
Please be sure, when submitting your hiring challenge, to include the script that you've used to create this Timemboard.
> My script is included in this pr.  [create_timeboard.rb](https://github.com/unclebconnor/hiring-engineers/blob/master/create_timeboard.rb)
  UPDATE: SHOW SCREENSHOT

Once this is created, access the Dashboard from your Dashboard List in the UI:

* Set the Timeboard's timeframe to the past 5 minutes
* Take a snapshot of this graph and use the @ notation to send it to yourself.
> I ended up setting the timeframe back to a 5 minute window earlier in the hour because there was nothing displaying in my rollup chart.  I guessed that this had something to do with there not being much data for the hour that I took the screenshot, despite the fact it should just be rolling up the average for the last hour.  I'm curious what values it's actually trying to calculate.  
> I tried to add an annotation to the heatmap but that function only seemed to work on the line chart.  Adding the annotations on that graph worked fine and I got the automated email from tagging myself.
  UPDATE: REWRITE AS DIRECTIONS
  
Pictured:
1. "Zoomed in" timeframe for a 5 minute window
2. Annotations
  
  ![Zoomed In Charts](https://github.com/unclebconnor/hiring-engineers/blob/master/images/07_5-minutes.png)
  ![Annotations](https://github.com/unclebconnor/hiring-engineers/blob/master/images/08_annotations.png)
  
* **Bonus Question**: What is the Anomaly graph displaying?
> The anomaly algorithm decides what is "normal" based on trend data and lets a user know when a host is behaving atypically.  Obviously, my hosts haven't existed long enough to provide meaningful data so I have to assume that "normal" was probabily defined by standard deviations or quartiles or a similar construct, and anomalies were values outside of that range.

---

## Monitoring Data

Since you’ve already caught your test metric going above 800 once, you don’t want to have to continually watch this dashboard to be alerted when it goes above 800 again. So let’s make life easier by creating a monitor.

Create a new Metric Monitor that watches the average of your custom metric (my_metric) and will alert if it’s above the following values over the past 5 minutes:

* Warning threshold of 500
* Alerting threshold of 800
* And also ensure that it will notify you if there is No Data for this query over the past 10m.
> The image below shows the settings I implemented for this task
  
  ![Monitor Alert Settings](https://github.com/unclebconnor/hiring-engineers/blob/master/images/09a_alert-settings.png)  
  
Please configure the monitor’s message so that it will:

* Send you an email whenever the monitor triggers.
* Create different messages based on whether the monitor is in an Alert, Warning, or No Data state.
* Include the metric value that caused the monitor to trigger and host ip when the Monitor triggers an Alert state.
* When this monitor sends you an email notification, take a screenshot of the email that it sends you.
> The image below shows one of the resulting emails  
> *my host.name tag came up with a blank value because I set this monitor up for both of my hosts.  In a real scenario, I'd likely be creating this kind of monitor for each host separately so that tag would work.  In a summary monitor like this, it would make more sense to use a general message such as "the system needs your attention", etc.
  UPDATE TO SHOW A SPECIFIC HOST
  
  ![Monitor Alert Email](https://github.com/unclebconnor/hiring-engineers/blob/master/images/09b_alert-email.png)
  
* **Bonus Question**: Since this monitor is going to alert pretty often, you don’t want to be alerted when you are out of the office. Set up two scheduled downtimes for this monitor:

    * One that silences it from 7pm to 9am daily on M-F,
    * And one that silences it all day on Sat-Sun.
    * Make sure that your email is notified when you schedule the downtime and take a screenshot of that notification.
> The following two images show:
> * The alert downtime settings
> * The resulting email
  
  ![Alert Downtime Settings](https://github.com/unclebconnor/hiring-engineers/blob/master/images/09c_alert-downtime-settings.png)
  ![Alert Downtime Email](https://github.com/unclebconnor/hiring-engineers/blob/master/images/09d_alert-downtime-email.png)
  
## Collecting APM Data:

Given the following Flask app (or any Python/Ruby/Go app of your choice) instrument this using Datadog’s APM solution:

Provide a link and a screenshot of a Dashboard with both APM and Infrastructure Metrics.
> I had a pretty tough time with this exercise, which had more to do with my environment(s) than the difficulty of the task.  In my linux environment, I played with a few different combinations of versions of pip and python, python-dev, etc. and caused some problems for myself a few times by not using sudo.  It turns out most of my issues were permissions, even though my error messages led me in circles for a bit.  I implemented the simple flask app that was given in the exercise, which only produced minimal data as seen in the screenshot below.  
  UPDATE:  REVIEW THIS EXERCISE AND REWRITE AS DIRECTIONS
  
>I was also never able to get the Tracer Agent installed on my osx host.  I'm not sure if I didn't understand the syntax or if the directions were simply innacurate but that, I think, is the one piece missing from my ruby implementation.  I updated the github repo with that app to include the initializer.  [Body Map Project](https://github.com/unclebconnor/body_map)
  UPDATE: REVIEW GO INSTALLATION AND RETRY
  
  ![Dashboard](https://github.com/unclebconnor/hiring-engineers/blob/master/images/10a_dashboard.png)
  
Please include your fully instrumented app in your submission, as well. 
> I used the flask app that was given:

```
from flask import Flask
import logging
import sys

# Have flask use stdout as the logger
main_logger = logging.getLogger()
main_logger.setLevel(logging.DEBUG)
c = logging.StreamHandler(sys.stdout)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
c.setFormatter(formatter)
main_logger.addHandler(c)

app = Flask(__name__)

@app.route('/')
def api_entry():
    return 'Entrypoint to the Application'

@app.route('/api/apm')
def apm_endpoint():
    return 'Getting APM Started'

@app.route('/api/trace')
def trace_endpoint():
    return 'Posting Traces'

if __name__ == '__main__':
    app.run()
```    

* **Bonus Question**: What is the difference between a Service and a Resource?
> A `service` refers to a feature set (to quote the manual) such as a web app or database.  `Resources` are queries within services, such as an API call (route) or SQL query.  So, for example, if I used a get route on my home page '/queryme' to access a database 5 times, I would be accessing 2 services (my app/home page and the database) via 10 resources (5x api calls and 5x database queries).

---

## Final Question:

Datadog has been used in a lot of creative ways in the past. We’ve written some blog posts about using Datadog to monitor the NYC Subway System, Pokemon Go, and even office restroom availability!

Is there anything creative you would use Datadog for?
> * It would be interesting from a design perspective to not only see the frequency of pages are clicked on a website (using APM to monitor routes?) but also the sequence and how long it takes a user to get to particular areas of interest.  Creating some kind of map could potentially help design teams evaluate their effectiveness.
> * It would also be interesting to see what types of devices users were accessing a website or specific features of a site from, in order to inform design decisions.  For example, if 80% of my traffic comes from mobile phones via Facebook links, I might focus more on mobile development.  

---
**Thanks for  this experience!**  
**I am grateful for any feedback you're willing to share, answers to questions, etc.**
