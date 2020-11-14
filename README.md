# FastSensor Dashboard Critique

FastSensor Dashboard is a web application that uses the FastSensor API to get alerts from ADAM and display them in three different charts. A line chart, a calendar and a heatmap. The technology stack is PHP to serve the website and the web pages use HTML, CSS and JavaScript. Some third-party libraries used are Bootstrap, Material for the layout, chart.js for the visualization, moment.js to work on dates, daterangepicker.js for the date picker, i18n for internationalization and stripe, even though there are no payment options in the dashboard. I will evaluate this website from three different angles, security, performance and UI/UX.

## Security

The user must be previously registered in order to display the data. Only authorized user can see the dashboard, to this effect the main page is a login page. We have a standard login page with a form that has a username and password field to authenticate the user via an HTTP request to the server.

Here is the main security concern. The authentication request is using the HTTP verb GET. Therefore, the password and username are in the URL. Even though there is a token passed in the request, it only protects against Cross-Site Request Forgery (CSRF). The credentials are in plain text in a URL which means it will be stored in the browser history, the server logs and probably by all third party programs that work with the user’s browser or network, such as browser extensions, VPNs, Firewalls…  With that said we understand that the credentials will be stored in several locations, not controlled by the user or FastSensor, where they could be stolen.

A solution would be to do the authentication request using the HTTP verb POST. That way the credentials will be in the request payload and that is not commonly logged by third parties or stored in the browser history/logs.

On another note the protection against CSRF could be done with the PHP session cookie, “PHPSESSID” in this case. By setting the parameters HttpOnly to true and SameSite to Lax it would achieve something similar to what the token does. With those parameters, the cookie could not be accessible by JavaScript and only added to the request originated from third parties on “safe” requests, such as GET, HEAD and OPTIONS.

Once authenticated a bearer token is stored in the HTML and used to authenticate the user for subsequent API calls.
It is worthy to note that the stripe library tracks user clicks and URL visited. The request URL is https://m.stripe.com/6 and the payload is some base 64 encoded data. Nonetheless it is something to keep in mind when this library is updated to monitor what else is this script doing.

## Performance

The performance of this web app is good, the visualization loads from connection to dashboard in 3.5 seconds. The common metric for load time is to be under 3 seconds according to Google, and the app actually achieves that, because the time for the dashboard to load without authentication is of 3 seconds. But it could be faster, it seems the application does way too many API calls to FastSensor GET alerts than it really needs.

At initialization there is 3 API calls to GET alerts with three different start and end date parameters. It would be more suitable to just do one call with a range from start date to end date that includes those of the 3 calls especially since there is a lot of overlap already. One call has a time range of a week ending now, another is a week ending yesterday and the last one is a week ending a week ago. One call of a range of 15 days until now would gather all the data needed for the charts. The three calls resolve in a total of 1469 ms. By just doing one call we could cut that time to a third and improve total loading speed by ~30%.

There are other superfluous API calls to GET alerts. When the user clicks on “View Report”. There is a request in order to generate the data table that is used to create the line chart. This data has already been loaded previously to create the chart in the first place. There is no reason not to keep this data in the front end and display it to the end user on demand without making a new request to the server.

A similar behavior happens with the heatmap. When the user selects a particular event from the dropdown list. There is an API call to GET alerts and in the selector parameter we see that the event id is added after the value ADAM (https://appsrv.fastsensor.us:8890/v1/locations/204D5F25/alerts?start_date=2020-11-08&end_date=2020-11-14&selector=%5BADAM%5D%3B00demo000013). The response of the API is the same that was sent without the added event id to the selector parameter (https://appsrv.fastsensor.us:8890/v1/locations/204D5F25/alerts?start_date=2020-11-08&end_date=2020-11-14&selector=%5BADAM%5). Therefore, we can see that there is an erroneous use of the API but more importantly there is no need to make this call, because the data has been loaded when initializing the heatmap. Once again, the data could have been stored and used again to filter between events and reload the heatmap.
According to the API documentation a range of 92 days can be queried. Making a default API call of ~61 days ending now at load time and storing this data in the front end would be enough to not make anymore request for all the options of the dropdown list, except “Custom”.

On top of loading the dashboard faster and making the transition between data range smoother, it would reduce drastically the amount of calls to the API.

## UI/UX

The design of this web application follows the material design style components. We have organized content using components like cards, charts, and tables. Navigation drawer for navigation and inputs and selection controls for user actions. Smooth communication with pop ups to alert the user of important messages and icons that provide key information. 

From the UI perspective there are small details that could be improved. The navigation drawer and the burger icon have the same function. The burger should only appear in mobile mode. Inspecting the code, the drawer is hidden when width is smaller than 992px (@media(min-width:992px)). This should be the point where the burger appears to replace the drawer.
In the calendar visualization the color gradient is too subtle. It is hard to differentiate greens that are next to each other. More contrast should be added between the colors to improve user readability.

Also, I noted that the footer link “GTI Technologies Inc” links tot the FastSensor home page and not the GTI Technologies website.
From a UX perspective, the dashboard seems to lack information at first glance. The first 2 charts have titles but not the last one. There are help bubbles for each chart, “What is this report”, to better understand each visualization but I feel like some of this data could be put forward in a more permanent way. Also, displaying the time period for each chart would be a nice information to have when analyzing this data.

Another feature would be to not logout on reload or keep the session active rather than demand credentials at every visit.


In a nutshell there is a security concern regarding the authentication procedure, the performance could be improved by making less API calls and the UX/UI could improve with a couple a simple display changes.
