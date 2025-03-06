# Hands-On Lab: Orchestration with Apache Airflow

In this lab, we’ll build and deploy Airflow data pipelines that pull, transform, and analyze cryptocurrency pricing data. You’ll get very familiar with Airflow DAGs and their tasks, as well as some good practice with Python's API and database integrations. Heck, you'll even get to play with a Slack bot for notifications. By the end, you’ll have several functioning pipelines and some _serious_ Airflow cred. 

But before we jump in, here are some important orienting notes:
1. Airflow is a fairly resource-hungry environment to run locally. You should be able to get away with running everything (if a little slowly) by providing 4GB of RAM to Docker, but ideally you should give it more like 8GB. (If your laptop only has 8GB of RAM, I'd recommend setting your docker RAM limit at 4GB.) Whether you do 4GB or 8GB (or more, if you've got enough to spare), I would also set Docker's memory swap settings up to 8+ GB.
2. As I mentioned in class, it's possible that some of you simply won't be able to get this environment running sucessfuly on your laptop. And that's okay! I've structured the assignment intentionally to allow you to get most of the work done on your own laptop without even needing to launch the full Airflow environment. (All you'll need for the first 50-70% of the assignment is a postgres database container.) So you can forge ahead and get Tasks 1, 2, and 3 finished on your own computer, then you can just do Tasks 4 and 5 in the lab if you need to. (Reach out if you have questions about this.)

Okay. Let's get to it.

---

## Task 1: Get the Environment Up and Running

First, let's make sure you have a working development environment. (For help working through this initial setup proces, you might also use the [lecture recording](https://www.dropbox.com/scl/fi/i44rww220il9gpvdmjl8t/2025-03-04-Lecture-14-Airflow-Day-2.mov?rlkey=nchsb7fmscgai4r7ooskdrbko&dl=0) from our in-class walkthrough.)

---

### 1.1 - Obtain and Set Up Your API Key

Head over to the [CoinCap API Key page](https://coincap.io/api-key) and grab a free API key. (You can use a throwaway email if you prefer.) CoinCap’s API can be used without a key, but using one ensures higher rate limits and stability​, and it will hopefully help us avoid getting blacklisted while we're all working on this assignment. Once you have your key, open the `crypto_tasks.py` file in the `dags/crypto_utils/` folder and paste the key into the designated spot (i.e., assign it to the `API_KEY` variable). This key will be used to authenticate your requests to the CoinCap API.

---

### 1.2 - Create and Activate a Python Virtual Environment

Create a `Conda` environment for this project (I used python 3.12.). After activating, install the python libraries found in `requirements.txt`. 

Now you can make sure you're ready to go by running the test script (`test_crypto_tasks.py`) to verify everything is set up correctly. This test script just runs some simple unit tests for the crypto pipeline functions found in `crypto_tasks.py`, and you'll be adding additional tests over the course of the first 3 tasks. 

If your setup is correct, the tests should run without errors and output the fetched crypto data and successful cleaning operations. It should look something like the screenshot below.

<img src="screenshots/readme_img/initial_test.png"  width="80%">

> [!IMPORTANT]
> 📷 Once the test script runs successfully, take a screenshot of your terminal showing the output (e.g., the test results). Save this screenshot as `screenshots/task_1.2.png`.

--- 

### 1.3 - Launch the Airflow Environment

Alright. [Hold on to your butts.](https://www.youtube.com/watch?v=UjvGAYuWSUA). It’s time to start your Airflow environment using `docker compose up -d`, run from the repository folder. In theory, all the necessary containers (including a database and Airflow web UI) will start. This process may take a while, but it shouldn't be extremely long (like more than 10 minutes). If it's taking longer than that (or if you hit errors that you can't resolve), reach out for some help. I don't want you getting stuck here. Again, you can accomplish all of Tasks 2 and 3 even without the full Airflow environment running. So let's chat if necessary.

After the system finishes booting (which _will_ take longer than the terminal will make you think; remember - you can look at the webserver logs and watch for the "Listening on 0.0.0.0:8080" message). Once you see that message, navigate to http://localhost:8080 in your browser. You should see the Airflow web interface, where you can log in with username/password: airflow/airflow. 

In the Airflow UI, you can filter to the `is566` tag to remove all the example DAGs from view, which should make it wasy to locate and open the DAG named “CryptoPrint”. Run the DAG manually (clicking the play button ►). You can then watch it run with glee until you see all the boxes turn dark green, indicating a successful run of all three tasks in the DAG. You should see something similar to my screenshot below.

<img src="screenshots/readme_img/first_dag.png"  width="80%">

Once you see that all tasks succeeded, CONGRATULATIONS – your Airflow instance is up and running! You’ve verified that Docker and Airflow are configured properly. 

> [!IMPORTANT]
> 📷 Take a screenshot of the Airflow UI showing the successful CryptoPrint DAG run (all tasks with green squares). Save it as `screenshots/task_1.3.png`.

--- 

## Task 2: Write Asset Table I/O Logic 
Now let's start building the actual data pipeline logic. We’ll write Python functions to interact with the cryptocurrency API and the local Postgres database. To make sure they are working as intended, we'll call them from a testing function, all without having to work through the Airflow system (yet). 

---

### 2.1 - Implement `load_asset_data()`

Open the `crypto_tasks.py` file and locate the (skeleton of the) `load_asset_data()` function. You'll need to complete this function so that it takes the cleaned crypto data (returned from the `clean_asset_data()` function) and inserts it into the `assets` table in the Postgres database.

As you figure out this logic, here are some important things to consider:
- the only portion of the skeleton function code that you'll need to change is between the `cursor = connection.cursor()` and `cursor.execute(postgres_insert_query, record_to_insert)` lines. Basically, you're looking for the right assembly logic to fill out the select statement string.
- I'll just point out my use of the `host` parameter, which has a default value of `postgres` (which is the hostname of the database server during runtime execution of the DAG), but which you can override with `localhost` when running the function in the `test_crypto_tasks.py` script. (You'll see that I've provided this command in the test script as a comment that you can un-comment.)
- Also pay attention to how I'm using `logging.info()` function calls to provide status information. Using this (instead of `print()`) is how you can ensure that your messages will be logged properly when you're running this code from within Airflow. You may want to use this same approach in other functions you write to help debug or monitor a DAG's progress.
- Feel free to provide the skeleton function, the table structure (found in `sql/01_create_db.sql`) and a sample structure of the cleaned crypto data to ChatGPT to make quick work of this logic. Just make sure you understand what's going on with the result you get back.

After making your edits in `crypto_tasks.py`, you can un-comment the line in `test_crypto_tasks.py` and then re-run the test function. If everything is working, you should see some new console output from the database insertion, like the screenshot below.

<img src="screenshots/readme_img/first_insert.png"  width="80%">

> [!TIP] 
> For this task, if you see console output like the screenshot above, you're good to go. But for some future tasks, it might be useful to have access to the database admin portal to check your inserts, troubleshoot, etc. To help with this, the docker compose includes an "Adminer" container, which you can access in the browser window at http://http://localhost:8081. There, you can provide the credentials that you see in the image below to login to the database admin portal.
>
> <img src="screenshots/readme_img/adminer_creds.png"  width="80%">
>
> The crypto data tables we're using are nestled under the `cryptodata` schema. If you poke around there, you'll be able to check that the asset record you just inserted is there, as in the screenshot below.

Okay. Back to the task. Assuming you got things working, you can take a screenshot and move on.

> [!IMPORTANT]
> 📷 Take a screenshot of the console output showing the successful test run, including "1 record successfully inserted into table" at the bottom. Save it as `screenshots/task_2.1.png`.

--- 

### 2.2 - Implement `compare_recent_crypto_record()`

This next function will calculate the percentage change in price for a given asset compared to its last recorded price in our database. 

> [!TIP]
> For this and all remaining functions we'll be creating, I'm going to provide a function definition "stub" that you can just paste into the `crypto_tasks.py` file. These stubs will clearly define what the function does, what its input parameters are and what it returns at the end. These stubs can be very useful when you're working with ChatGPT to fill in logic, but I'll again remind you that you want to be intentional with your prompts so the logic doesn't get away from you.

The function is designed to be used as a part of the asset price monitoring pipeline to provide some context about how much the price of a given asset has changed since it was last updated in our local system. TO make this comparison, the function will take in an asset_id and price from some newly cleaned asset data (which will be inserted into the asset table in a subsequent task). The function will run a query to get the most recent pricing entry for the asset, do some simple math to calculate the percent change between the price retrieved from the asset table and the new price, and assemble the comparison results in a dict return object. 

> [!TIP]
> In case it's helpful, here's the math for the percent change: `percent_change = ((new_price - previous_price) / previous_price) * 100`

One important thing to be aware of: cryptocurrency prices are tracked with extreme precision (16 decimials), which is more precise than standard `float` numbers can handle. You'll notice that I've imported a couple utilities from the `decimal` package to help with this precision. (The `getcontext()` function call at the top of the script sets the precision for the whole script to 20 decimal places.) Anyway, this will be relevant for you when you are getting ready to do math to compare the price values: you should cast the two price values using something similar to `previous_price = Decimal(previous_price)`. Doing so will make for more meaningful percent changes (which could otherwise just show as a zero without the `Decimal` precision).

Okay. Here's the function stub. Get coding!

```python
def compare_recent_crypto_record(asset_id, new_price, host='postgres'):
    """
    Queries the most recent priceUsd and timestamp for a given asset_id from the assets table.
    Computes the percent change from the existing price to the new_price.
    
    Args:
        asset_id (str): The crypto asset ID to query.
        new_price (float): The latest price to compare against the most recent record.
        host (str): The database host (default: 'postgres').

    Returns:
        dict: {
            "previous_price": float,
            "previous_timestamp": str (YYYY-MM-DD HH:MM:SS),
            "percent_change": float
        }
        or None if no records exist.
    """
```

When you have it working, you can add a new test case to the test script to call this function as a part of your test flow. (Note that in order to provide a meaningful comparing, `compare_recent_crypto_record()` needs to be called after cleaning but _before_ loading an asset's pricing information.)

A successful test should look something like my screenshot below (remember to add some `logging.info()` output in your function so you can see something when it runs).

<img src="screenshots/readme_img/comparison_done.png"  width="80%">

> [!IMPORTANT]
> 📷 Take a screenshot of the console output showing the comparison function running successfully. Save it as `screenshots/task_2.2.png`.

--- 

### 2.3 - Configure Slack Notifications

Time to add some fun Slack notifications! Below, I'll provide a `send_slack_message()` function that uses a simple Slack webhook to post a message to a dedicated channel in our class Slack space. You can paste this into your `crypto_tasks.py` file without modification: 

```python
def send_slack_message(message,
                       SLACK_WEBHOOK_URL='https://hooks.slack.com/services/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'):
    """
    Sends a message to the AirflowBot in the #airflow-chatter channel.
    """
    if message is None:
        logging.error("Received None for message. Skipping send.")
        return
    
    payload = {"text": message}
    response = requests.post(SLACK_WEBHOOK_URL, json=payload, headers={'Content-Type': 'application/json'})

    if response.status_code == 200:
        logging.info("Message successfully sent to Slack!")
    else:
        print(f"Failed to send message: {response.text}")
        response.raise_for_status()
```

Now you just need to write a function that will receive the comparison data we generated in Task 2.2 and construct a message that reports the result of the comparison to the channel for everyone to enjoy.

The function will use the newly cleaned asset data from the API and the result of your comparison function. The only requirement is that the message contain all of the following: 
- The asset's actual name (not `asset_id`)
- The newly obtained price, formatted into USD
- The (formatted) timestamp of your newly obtained asset pricing data
- The previous price from the comparison, formatted into USD
- The (formatted) previous timestamp form the comparison
- The percent change (formatted as a percent) from the comparison.

A bland version of a formatted pricing update message might look like this:

> **Bitcoin** is currently priced at **$89,615.92** USD (as of 2025-03-06 07:15:41).
> This is a **-0.26%** difference from my last recorded price ($89,849.04 on 2025-03-06 07:00:09).
> With ❤️, Rogue 1

Other than the above requirements, you can format your message in whatever way you want, so feel free to have fun with this. You don't need to sign it with your real name if you don't want to. (But please don't abuse this ability to post anonymous messages to the whole class...this should be _harmless_ fun.)

If you provide a good log entry in your formatting function, you'll be able to iterate quickly to get the formatting right, viewing the result right in the console. (That would be much better than testing your formatting with a bunch of messages sent to the Slack channel!)

Alrighty. Function stub, coming in hot:

```python
def format_slack_message(asset_obs, comparison_result):
    """
    Formats a Slack message with asset name, current price, timestamp, and comparison to the last recorded price.
    
    Args:
        asset_obs (dict): The latest asset data including name, price, and timestamp.
        comparison_result (dict): The result from compare_recent_crypto_record(), including previous price, timestamp, and percent change.

    Returns:
        str: A formatted Slack message.
    """
```

When you have everything looking lovely, you can then add the `send_slack_message()` function to the  test script and share your message with everyone. You should then capture a screenshot like the one I've included below, with your console window alongside or overlayed with your slack window, showing that your console output matches (more or less) the message you sent to the slack channel.

<img src="screenshots/readme_img/slack_message.png"  width="80%">

**WAIT!** Before moving on, go ahead and comment out your `send_slack_message()` function call in your test script. We've got more testing to do, and we don't want to add a bunch of noise to the channel with our subsequent tests.

> [!IMPORTANT]
> 📷 Take a screenshot of the console + Slack window as described above. Save it as `screenshots/task_2.3.png`.

--- 

## Task 3: Implement Historical Pricing Analysis Functions

Why stop with one data pipeline when we can build two? This second one will be a bit less nuanced. It mostly consists of a second set of ETL tasks that pull historical asset pricing data from the same API service, clean up that data, and load it into a separate table. I've included this second pipeline so that you have the opportunity to complete these similar-but-different tasks with less guidance from me as a way to demonstrate that you've learned some good things in the previous task. We will be creating three functions to pull, clean, and load one year’s worth of daily pricing data for a given asset, then use the data with some light analysis functions.

Like Task 2, this task will be completed exclusively within the `crypto_tasks.py` and `test_crypto_tasks.py` files. You'll write the functions, include a test call in the test script, capture a screenshot of the result in the console, and move on.

---

### Task 3.1 - Implement `pull_historical_asset_pricing()`

This function will make a slightly different API call to the same API service that we used in the previous task. This means that you can get most of the function written by copying the content of the `pull_asset_data()` function and modifying it. The primary modifications include:
- Use the API documentation [here](https://docs.coincap.io) to assemble a different URL string. You'll find the format and potential parameters under the `GET /assets/{{id}}/history` heading on that page. In fact, the example on that page (with the `interval=d1`) is essentially the exact URL you'll be using (though you'll fill in the `asset_id` dynamically).
- Because of the way the data is returned from this endpoint (it doesn't actually include the `asset_id` that was used to request the data), you'll have to manually assemble the return object to include the `asset_id`. The return object will be a dict with 3 top-level items (`asset_id`, `data`, and `timestamp`), as shown in the function stub below.
- Lastly, because the data will come back with 365 days' worth of data, it's sort of annoying to write a log message. So I'll just provide the `logging.info()` command for you here:
> ```python
> logging.info(
>    f"Historical API Response (first 3 records): \n", 
>    f"{json.dumps({ "asset_id": historical_data["asset_id"], "data": historical_data["data"][:3], "timestamp": historical_data["timestamp"] },indent=2)}"
>  )
> ```

> [!TIP]
> One more pro tip: You can create a free account on Postman.com to experiment a bit with API calls. In fact, the CoinCap API documentation has a handy "Open in Postman" button that will populate their documentation functions in a Postman workspace for you, which is an excellent way to test and get familiar with how the data comes back. The screenshot below shows my test of the history endpoint.
> <img src="screenshots/readme_img/postman.png"  width="80%">

Okay. Use the suggestions above to fill in the function stub below. Then add a test function call in the test script, and capture and save a screenshot.

```python
def pull_historical_asset_pricing(asset_id):
    """
    Queries the API for historical daily pricing data of a given asset.
    
    Args:
        asset_id (str): The crypto asset ID to query.

    Returns:
        dict: A dictionary containing the asset_id and its historical pricing data.
    """
```

Here's my console after running the test script with this function added:

<img src="screenshots/readme_img/historical_pull.png"  width="80%">

> [!IMPORTANT]
> 📷 Take a screenshot of the console output showing the historical data pull function running successfully. Save it as `screenshots/task_3.1.png`.

--- 

### 3.2 - Implement `clean_historical_asset_pricing()`

The raw data from CoinCap needs to be cleaned up a bit. To keep things exciting (and set up a cool learning opportunity at the end of the lab), this time we'll be converting the data to a pandas dataframe. 

This is pretty straightforward, actually. You can extract the `asset_id` and `data` elements from the raw historical data, simply cast the `data` as a `pd.DataFrame`, and then do some column formatting before returning the dataframe. (And a simple way to add a log message would be to use the `.head()` function on the dataframe to send a little preview to the log/console.)

> [!IMPORTANT] 
> The `time` column in this dataframe is an integer representing an epoch timestamp, and it's important that you leave that column unconverted. It's likely that ChatGPT will want to convert it for you, but don't let it! The `date` column that originated from the API already provides a human-readable date. Leave the `time` column as an integer so that we can use it later as such.

Okay. Go to it. You'll be done in no time. Here's the function stub:

```python
def clean_historical_asset_pricing(historical_data):
    """
    Cleans raw historical asset pricing data and returns a pandas DataFrame.
    
    Args:
        historical_data (dict): The raw JSON data from the API.

    Returns:
        pd.DataFrame: A DataFrame with columns ['asset_id', 'priceUsd', 'time', 'date'].
    """
```

Get the function working, then take a screenshot of the pandas preview in the console, sorta like this one:

<img src="screenshots/readme_img/historical_clean.png"  width="80%">

> [!IMPORTANT]
> 📷 Take a screenshot of the console output showing the comparison function running successfully. Save it as `screenshots/task_3.2.png`.

--- 

### 3.3 - Implement `load_historical_asset_pricing()`

Now we just load the historical data into the `historical_asset_pricing` table in the database. Again, this load function will be largely the same as the `load_asset_data()` function from Task 2, so I would start by copying the logic from that one and the modifying it as needed. For this function, you'll need to assemble the query from a dataframe instead of a dictionary. You'll also be doing a much larger insert (with 365 records instead of just one. If you poke around a bit online (and/or ask Chat for some syntax help), you'll find that you can use the `cursor.executemany()` approach to actually do a large insert with a single command. That requires you to assemble a "list of tuples" from your dataframe, which can then be passed as a single object to fill in the `VALUES` in the insert query.

Before you rush to write your code, though, I'll just note that there is a composite primary key constraint applied to this `historical_asset_pricing` table (which you can examine in the `sql/01_create_db.sql` script). This is to prevent a lot of duplication for a given asset's pricing history if you happen to run the query for the same asset more than once while testing. This means that you'll need to account for this in your query so that the database doesn't complain about duplicate primary keys.

> [!TIP]
> Did you know that Postgres doesn't support `UPSERT`s? I didn't either until I write this assignment. Instead you can look into the `ON CONFLICT` functionality.

Got it working? Good. You should produce some nice log messages and see them in the console like my screenshot below when you have the tests running appropriately.

<img src="screenshots/readme_img/initial_posts.png"  width="80%">

> [!IMPORTANT]
> 📷 Take a screenshot of the console output showing the historical data having loaded successfully. Save it as `screenshots/task_3.3.png`.

--- 

### 3.4 - Quick Test of Two Analysis Functions

Don't worry! That last function was the last one you had to write. Phew! But to demonstrate a couple cool things in the final tasks, we needed a couple of "analysis" functions that will run some queries on the historical data after it's loaded. But since I'm sure you're tired of writing python code at this point, I'll just give them to you.

I've placed them in `crypto_utils/analysis_functions.py`, so you'll just need to import them into your test script and then call each of them. Doing so should produce some output similar to the screenshot below. Easy peasy.

<img src="screenshots/readme_img/analysis_functions.png"  width="80%">

> [!IMPORTANT]
> 📷 Take a screenshot of the console output showing the analysis function running successfully. Save it as `screenshots/task_3.4.png`.

--- 

## Task 4: DAGs for the Assets Data

Okay! We're finally ready to start using Airflow. If you have been able to run Airflow in the background this whole time while you've been doing the other tasks, you can proceed to build out and test the DAGs described in these final two sections.

If you have had trouble getting Airflow to run reliably, now would be the time to save all of your working code, commit and push your changes, and then head to the lab.

Ready? This is going to be fun.

---

### 4.1 - The `CryptoPostgres` DAG

The first one is easy. You already know that the `CryptoPrint` DAG runs (in Task 1). This first DAG will almost be an exact copy of that one, except that you'll be making use of your spiffy new `load_asset_data()` function to add an entry to the database rather than simply printing the cleaned data to the console/logs.

So make a copy of the `01-CryptoPrint.py` file (named `02-CryptoPostgres.py`), change a few names, and (most importantly) swap out the print task for the load task. 

> [!TIP]
> In most cases, when you are adding or modifying the DAG files, you don't need to spin down the whole system. Airflow is going to monitor that folder and you should be able to refresh and see you new dag appear. This is also true for changes you need to make. And you can always examine the code that Airflow thinks it's going to run by looking at the DAG details and clicking on the Code tab.

Once you have your DAG ready, you can run it with the "Play" arrow, and watch as you (hopefully) get to see three green boxes like in my screenshot below. Yay!

<img src="screenshots/readme_img/dag2.png"  width="80%">

> [!IMPORTANT]
> 📷 Take a screenshot of the Airflow DAG interface showing that your new DAG ran successfully. Save it as `screenshots/task_4.1.png`.

--- 

### 4.2 - The `CryptoSlack` DAG

Time for some more fun with Slack! Make another DAG (call it `03-CryptoSlack.py`) where you can create a new, more complex DAG that will make use of all of the rest of the functions we built for the asset data.

Specifically, you're going to have a total of 5 tasks that do the following:
1. Pull the asset data for a given asset_id (similar to the first two DAGs)
2. Pull the asset data for a given asset_id (similar to the first two DAGs)
3. Load the asset data for a given asset_id (similar to the DAG from 4.1)
4. Use the `compare_recent_crypto_record()` function to pull comparison information between the current price and the last updated price from the database.
5. One task that uses both the `format_slack message()` and `send_slack_message()` functions to send your messages to the Slack channel.

Before you launch your new DAG, though, pay attention to the dependencies established at the bottom of the DAG. The purpose of this DAG is to help you think very carefully about the order in which these tasks need to be accomplished, and to use the logical flow of the tasks and their returned results to establish the correct order. (Think in particular about when the comparison task needs to run and how to establish the proper sequence.)

You may have to get a bit tricky with the dependencies to make sure that, for example, the comparison happens before the new data is loaded. This is easier when the functions have return statements because then Airflow will recognize that as a step that needs to be completed before the next one. In the case of the Slack and load tasks (where you probably aren't returning anything), Airflow is going to forge ahead and potentially get them out of order. There are a few different ways to be explicit about this, but the easiest way is probably to create a return statement even if it's not really necessary. That gives you an "event" to work with that you can then incorporate to establish the proper dependency and order of your tasks.

You can always check to see the effect of your changes on the dependency order by looking at the "Graph" tab in the DAG details (see the screenshot below). This will show you what Airflow has understood as the order of dependencies, and will give you something to check as you play around with different approaches.

<img src="screenshots/readme_img/slack_dependencies.png"  width="80%">

After you get your DAG graph looking like mine in the screenshot above, you can also make sure it runs with green boxes, and then save a screenshot. (It'll probably also post a message to Slack, but you don't need to capture that for now.)

> [!IMPORTANT]
> 📷 Take a screenshot of the DAG graph page showing that you got the dependencies right and that your DAG runs successfully. Save it as `screenshots/task_4.2.png`.

--- 

### 4.3 - Schedule your `CryptoSlack` DAG

Just for fun, go ahead and make a change to one line of the `03-CryptoSlack.py` file to setup it up to run every 30 or 60 seconds. You'll accomplish this simply by changing the `schedule_interval` parameter in your `@DAG()` decorator definition to be something other than `None`. You can do a once-a-minute schedule using some cron job syntax (which has a maximum granularity of minutes), or you can use a timedelta operation to get it down to every 30 seconds. Your choice.

Once you save your DAG and Airflow recognizes that it's scheduled, it'll probably start firing off. Let it run for a few minutes so you can show that the logic is all working properly (especially that there are different percentage change values each time it fires). Then (and this is **VERY IMPORTANT**), please deactivate the DAG. 🙂 You can do this using the toggle in the Airflow interface (or I guess you could revert your change in the python file to remove the schedule).

Again, just to reiterate, **please don't leave your DAG running** while you move on. We don't want Slack to blacklist us.

You may need to do some searching and filtering in Slack if lots of people are working on this at the time you had your DAG running, but hopefully you can snag a screenshot sort of similar to mine below that shows that you've let your DAG run on a schedule for a bit.

<img src="screenshots/readme_img/scheduled_dag.png"  width="80%">

> [!IMPORTANT]
> 📷 Take a screenshot of the `#airflow-chatter` channel showing that your DAG ran 3-4 times on a schedule. Save it as `screenshots/task_4.3.png`.

--- 

## Task 5: DAGs for the Historical Data 

We're on the home stretch, people! As with the functions you wrote to pull, clean, and load the historical data, the purpose of this last task is just to add some reinforcement learning with even less oversight and handholding. I'll have you create a DAG that handles the historical data pipeline, and then we'll have a little bit of fun learning about how Airflow can process things in parallel.

---

### 5.1 - The `CryptoHistory` DAG

Ready to prove to yourself that you can build data pipelines on your own? Okay! Make a new DAG called `CryptoHistory` that pulls, cleans, and then loads the historical data for a cryptocurrency of your choosing. Create the necessary tasks and dependencies, deploy the DAG, and give it a test run. 

After you're done, you'll probably be looking at something pretty close to the screenshot below (though you are welcome to name your tasks whatever you'd like).

<img src="screenshots/readme_img/history_dag.png"  width="80%">

> [!IMPORTANT]
> 📷 Take a screenshot of the Airflow interface showing your CryptoHistory data pipeline in all its glory. Save it as `screenshots/task_5.1.png`.

--- 

### 5.2 - The `CryptoParallel` DAG

This task involves creating a slightly modified version of the DAG you just built that uses Airflow's horizontal scalability to load and analyze the historical data in parallel. The idea is to demonstrate Airflow’s ability to run independent tasks concurrently to speed up the pipeline. 

Because this is new (and FUN!), I'll give you some more detailed guidance. Your DAG will have four tasks:
1. `pull_and_clean_history_task()`: To make this parallel stuff work, you'll need to combine the pulling and cleaning of historical data into a single task. Remember that these tasks are just special functions, so you can absolutely call more than one of our crypto_utils functions within the same task. The task should return the cleaned historical data for the requested `asset_id`.
2. `load_history_task()`: this is the same task as you used in 5.1 above, so nothing special here. The task should receive cleaned historical data and load it into the table. 
3. `moving_ave_task()`: This task just runs the `calculate_7_day_moving_average()` function that we tested in Task 3.4. Note that you'll need to import these analaysis functions from the `analysis_functions.py` file to make them available to the DAG.
4. `annual_vol_task()`: Just like the previous one, this task is just a wrapper for the `calculate_annualized_volatility()` function from `analysis_functions.py`.

Go ahead and establish a logical dependency flow among these functions. Tasks 1 and 2 are pretty straightforward, but obviously the two analysis tasks can't start until the data load has finished. Again you can establish this dependency in a number of ways, but the easiest is probably to simply have the `load_history_task` return a value of some sort when it's done, Which you can then capture in the dependency flow. 

Once you have a DAG graph that looks like the one in the image below, you can just test it to make sure it runs. No need to take a screenshot of your own at this point, because we're not done! This is just a quick gut check to make sure you have everything lined up and working.

<img src="screenshots/readme_img/parallel1.png"  width="80%">

Alright, now we're going to have some fun. Within the same DAG file, we're going to convert the existing flow to one that will operate in parallel at key points in the workflow. It turns out that the API we're using has historical crypto pricing information available for a few thousand different currencies. We're not going to gather data for all of them, but we'll use this as an opportunity to demonstrate how to scale a data pipeline horizontally. 

Let's collect historical pricing data for the top 6 crypto currencies (in terms of market cap). Those currencies are contained in the list below, which you can paste into your script:

```python
asset_list = [
        "bitcoin", 
        "ethereum", 
        "tether", 
        "xrp", 
        "binance-coin", 
        "dogecoin"
      ]
```

Now we'll make a few modifications to the DAG logic to enable us to gather all 6 of those dataset simultaneously. Follow along:

First, change the way you are calling the `pull_and_clean_history_task()` task near the bottom of your script. (Don't change the task definition itself; it's already compatible with what we're going to do.) Whereas before you called the task with something like this: 

  > ```python
  >     hist_clean = pull_and_clean_history_task('dogecoin')
  > ```

To this:

  > ```python
  >     hist_clean_list = pull_and_clean_history_task.expand(asset_id=asset_list)
  > ```

Notice that we added a `.expand` before the parentheses, and we're passing the entire list of asset_ids to the task. Notice also that I'm naming the resulting object as `hist_clean_list`, which is to help us keep track of the fact that, when we do this parallel execution of all 6 pull_and_clean operations, the results are returned in a list structure that we'll have to deal with in a minute. 

Next, we'll make sure that the `hist_clean_list` gets passed to the `load_history_task()` (since we renamed it), and then we'll make a slight modification to the `load_history_task()` function. Whereas before, you were probably just receiving the incoming clean data and passing it to the `load_` function, the `hist_clean_list` is now being passed into that task as a _list of dataframes_ rather than a dataframe like before. So, up in the `load_history_task()` definition, we just need to add some logic to extract and combine all of those (nested) dataframes into a single, long dataframe. Because they are all formatted the same way with identical columns (since they all passed through the same cleaning function), we can just stack them together. The syntax for that is:

  > ```python
  >     together =  pd.concat(list(hist_clean_list))
  > ```

Just to walk through what happened above, `pd.concat()` is designed to do exactly the thing that we're trying to do, namely, stack together a bunch of dataframes with identical column structures. But inside of that function we have to explicitly convert `hist_clean_list` to a list because the `task.expand()` operation that we used earlier actually returns something called a "lazy" list, which is bascially a looser way of collecting things that may or may not be similar. Anyway, point is that we are (a) converting the lazy list of dataframes to a "real" list of dataframes that `pd.concat()` can then stack together into a long single dataframe that can then be passed to the load function because it's now just a regular dataframe like that function expects.

Pretty fun, right?

One last thing, just to demonstrate some additional parallel options. We are now collecting 6 different historical pricing datasets, so we can apply the analysis functions to any of them, and none of them have any interdependecies so they could run at the same time. 

Another way to do this is to just establish a bunch of simultaneous tasks using the regular Airflow dependency logic. To do this, just make 5 copies each of the _calls_ of the `moving_ave_task()` and `annual_vol_task()` tasks, providing each of the 6 asset_ids from the `asset_list` to a different task call. Because all of these tasks are similarly dependent on the load task finishing and nothing else, Airflow will allow them to run in parallel.

To confirm that you have this right (and to get ready to run everything with our new-found parallelization skills!), head to the Airflow interface and look at the graph for this DAG. It should look something like the screenshot below:

<img src="screenshots/readme_img/parallel2.png"  width="80%">

The `.expand()` functionality doesn't get represented graphically by Airflow (though you may notice that there are now square brackets next to the `pull_and_clean_history_task() []` on the left). But the dependencies implied by the many copies of our analysis functions are now fanning out in one big simultaneous level.

Okay. Ready for lauch? 3...2...1...go!

Hopefully that worked for you and you're seeing green squares. If so, you can snag a screenshot, pat yourself on the back, and call this one DONE.

Thanks for making it to the bottom. What a ride!

## Submission Instructions

Ensure you have completed all tasks and saved the following screenshots:
- `task_1.png`
- `task_2.png`
- `task_3.png`
- `task_4a.png`
- `task_4b.png`
- `task_5.png`

Commit all code and screenshots to your repository and push your changes. 
