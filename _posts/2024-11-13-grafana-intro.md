---
title: "Grafana: Quick Introduction"
last_modified_at: 2024-11-13T16:20:02-05:00
categories:
  - Grafana
tags:
  - Grafana
  - Observability
classes: wide
---

# Grafana

![Grafana Dashboard](/blog/assets/images/grafana-dashboard.png)


Grafana is an open source software, mostly used for visualization and analytics. 
It allows you to query and visualize data stored in external applications, acting like a presentation layer for your data.
This way it makes you understand your data and gain insights.  
Grafana is handling 3 types of data: metrics, logs and traces.

Grafana can do much more than visualizations, but we'll not consider them in this post.

# Grafana Concepts

To use Grafana effectively we need to understand its basic concepts first.

- **Plugin**: Extends the functionality of Grafana by connecting a data source, adding a new type of panel etc. 
- **Data Source**: Connects to an application, which holds the data you wanna analyze.
- **Panel**: A type of graphical representation of your data. Fetches data from a data source and visualizes it. You can query the data source, filter and transform the data etc. A graphical representation might be a type of chart, a table etc. In the image below you see a chart and its data traversing different stages: 
    
    ![Panel](/blog/assets/images/grafana-panel.png)

- **Dashboard**: A set of panels visualizing a different aspect of your data. Usually you organize a related group of panels into a dashboard to have a holistic view of your application/system. You can create as many dashboards as you want. You see a dashboard on the top of the page. 


Please find in the references section the links to documentation explaining these concepts in detail.

# Demo 

In the following we want to create a dashboard with a panel. The panel will display data residing in Prometheus.

But we've to install Grafana first. 
Please head over to [Grafana](https://grafana.com/docs/grafana/latest/setup-grafana/installation/) and follow the instructions for your OS. 
After you've installed Grafana, open the URL http://localhost:3000 in your browser. Once you've entered the default username and password, you'll see:

![Grafana Home Page](/blog/assets/images/grafana-home-page.png)


## Create A Dashboard

We'll create our first dashboard. Select the item `Dashboard` from the menu on the left. On the dashboards page, click on the button `New dashboard` to create a new dashboard, which will take you to the screen below:

![Grafana Create Dashboard](/blog/assets/images/grafana-create-dashboard.png)

Click on `Add visualization`. The new page requires you to select a `data source`. Please click on `Prometheus` as we want to connect to Prometheus to fetch data for our visualization. We should see something like:

![Grafana New Panel](/blog/assets/images/grafana-new-panel.png)

From the `Metric` drop down box we select our custom metric `home_temperature_in_celcius`, which we did create in a previous [post](/prometheus/prometheus-intro): 

![Grafana Panel - Custom Metric](/blog/assets/images/grafana-panel-select-metric.png)

We can apply filtering and transformations to the data by using the provided UI elements: 

![Grafana Panel - Query Tab](/blog/assets/images/grafana-panel-query-tab.png)

Edit panel properties like title, time frame etc. and save the panel by clicking on `Save dashboard` and give your dashboard a name like `Smart Home`:

![Grafana Panel Properties](/blog/assets/images/grafana-panel-properties.png)

Now, click on the button `Back to dashbarod` on top, then on `Exit edit`.

We did finish creating a new dashboard. Our new dashboard named `Smart Home` should look like:

![Grafana - New Dashboard View](/blog/assets/images/grafana-new-custom-dashboard.png)

We've only one panel in our dashboard. You're free to add more panels, which should relate to the subject of the dashboard - in our case `smart home`.
Grafana allows us to place the panels in the order we want using drag and drop etc. 

# Summary

We did introduce Grafana and talk about its basic components. After installing Grafana on our local machine we did create a custom dashboard consisting of a single panel.

## References

- [Grafana](https://grafana.com/grafana)
- [Grafana Plugins](https://grafana.com/grafana/plugins)
- [Grafana Dashboards](https://grafana.com/grafana/dashboards)