# Brian Connor's excellent adventure through the hiring exercise

## Notes on setup
* I was able to install vagrant and the datadog agent with no trouble
* I confirmed that the agent was reporting by looking at the host map page
* I also installed an agent in my osx terminal, hoping that I'd get data from two hosts to compare, and because I'm less familiar with the linux vm.  My host map page has changed a few times and at one point I had 3 hexagons, 2 of which represented my macbook.  From this [help article](https://goo.gl/Zm5rY4) It seems that there are cases where a host might send multiple unique names, which Datadog then aliases separately.  The problem seems to have resolved itself (after a few rounds of uninstalling and reinstalling).

---

## Collecting Metrics:
* Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.  
> I was able to successfully add tags on both the osx and linux hosts.  I had a bit of trouble figuring out how to edit text on Vagrant at first (I used vi) but eventually made it work.  There was a significant lag updating the UI from both hosts, but the VM took much longer.  I did a series of restarts for the agents and logged out/in a few times.  I'm not sure if those were necessary or if I just needed to be patient and I could anticipate a client being similarly confused.  I'm curious what the expected or recommended protocol is.  
  
  ![Tags Created Successfully](https://github.com/unclebconnor/hiring-engineers/blob/master/images/01_tag-example.png)
  
* Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.
> I don't know why it wasn't obvious at first to rename the "example" file, but once I figured that out the installation was smooth on both hosts, though I'm still running into a sort of annoying issue with permissions on vagrant.  I'm sure I'll figure that out by the time I'm done.
  
  ![Postgres Installed Successfully](https://github.com/unclebconnor/hiring-engineers/blob/master/images/02_Postgres-Install.png)
  
* Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.
> The environment(s) are starting to make much more sense.  I've only done some basics on python so had to check on some syntax, but otherwise this was fairly straight-forward.
  
  ![Agent Check Installed Successfully](https://github.com/unclebconnor/hiring-engineers/blob/master/images/03_agent-check.png)
  
* Change your check's collection interval so that it only submits the metric once every 45 seconds.
> On my linux host, I changed this via the yaml file
  
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
  
  Pictured below:
  1. The manually created timeboard
  2. The timeboard created via the API and the script
  
  ![Manual Timeboard](https://github.com/unclebconnor/hiring-engineers/blob/master/images/06a_timeboard-manual.png)
  ![Timeboard from API](https://github.com/unclebconnor/hiring-engineers/blob/master/images/06b_timeboard-from-api.png)
  
Please be sure, when submitting your hiring challenge, to include the script that you've used to create this Timemboard.
> My script is included in this pr.  [create_timeboard.rb](https://github.com/unclebconnor/hiring-engineers/blob/master/create_timeboard.rb)

Once this is created, access the Dashboard from your Dashboard List in the UI:

* Set the Timeboard's timeframe to the past 5 minutes
* Take a snapshot of this graph and use the @ notation to send it to yourself.
> I ended up setting the timeframe back to a 5 minute window earlier in the hour because there was nothing displaying in my rollup chart.  I guessed that this had something to do with there not being much data for the hour that I took the screenshot, despite the fact it should just be rolling up the average for the last hour.  I'm curious what values it's actually trying to calculate.  
> I tried to add an annotation to the heatmap but that function only seemed to work on the line chart.  Adding the annotations on that graph worked fine and I got the automated email from tagging myself.
  
Pictured:
1. "Zoomed in" timeframe for a 5 minute window
2. Annotations
  
  ![Zoomed In Charts](https://github.com/unclebconnor/hiring-engineers/blob/master/images/07_5-minutes.png)
  ![Annotations](https://github.com/unclebconnor/hiring-engineers/blob/master/images/08_annotations.png)
  
* **Bonus Question**: What is the Anomaly graph displaying?
> The anomaly algorithm decides what is "normal" based on trend data and lets a user know when a host is behaving atypically.  Obviously, my hosts haven't existed long enough to provide meaningful data so "normal" was probabily defined by standard deviation or something like that, and anomalies were values outside of that range.

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

* **Note**: Using both ddtrace-run and manually inserting the Middleware has been known to cause issues. Please only use one or the other.

* **Bonus Question**: What is the difference between a Service and a Resource?

Provide a link and a screenshot of a Dashboard with both APM and Infrastructure Metrics.

Please include your fully instrumented app in your submission, as well. 

## Final Question:

Datadog has been used in a lot of creative ways in the past. We’ve written some blog posts about using Datadog to monitor the NYC Subway System, Pokemon Go, and even office restroom availability!

Is there anything creative you would use Datadog for?
