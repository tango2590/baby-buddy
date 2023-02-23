<h1>BabyBuddy in HomeAssistant</h1>

This project, which revolves around the [BabyBuddy Project](https://github.com/babybuddy/babybuddy), is meant to highlight my own way of getting and posting data between the BabyBuddy Add-on and HomeAssistant. This project is ongoing, and I apologize for the lack of polish or random comments I left in Node-RED. Our first-born came 4 weeks early, so my timeframe on the project became hours instead of weeks. Please note that since this is my first child, I did not reference `child_id` at this time.

The lovelace is heavily inspired by @Noblewolf’s post, with modifications made to it: https://community.home-assistant.io/t/baby-buddy-dashboard-blueprint/488162
The automation side of things was initially inspired by both Noblewolf’s blueprint, as well as @OCT0PUSCRIME’s input helpers: https://community.home-assistant.io/t/wip-baby-buddy-integration-frontend/464123


<h2>What’s Required:</h2>

- [BabyBuddy HomeAssistant Add-on](https://github.com/OttPeterR/addon-babybuddy)
- [BabyBuddy Integration](https://github.com/jcgoette/baby_buddy_homeassistant)
- [Node-RED Add-on](https://github.com/hassio-addons/addon-node-red)
- [Stopwatch automation](https://community.home-assistant.io/t/stopwatch-with-start-stop-resume-lap-and-reset/443994)
- [custom:button-card](https://github.com/custom-cards/button-card)
- [custom:apexcharts-card](https://github.com/RomRider/apexcharts-card)
- [custom:multiple-entity-row](https://github.com/benct/lovelace-multiple-entity-row)
- Installation of Noblewolf’s blueprint *(found above)*
- Addition of a modified list of input helpers *(found below)*

<h3>List of Input Helpers</h3>

- [Input Boolean](https://www.home-assistant.io/integrations/input_boolean/)
  - Probiotic/Vitamin D
- [Input Button](https://www.home-assistant.io/integrations/button/)
  - BB Feeding Helper
  - BB Pumping Helper
  - BB Diaper Helper
  - BB Sleep Helper
  - BB Tummy Time Helper
- [Input Text](https://www.home-assistant.io/integrations/input_text/)
  - BB Notes
- [Input Number](https://www.home-assistant.io/integrations/input_number/)
  - BB Amount
- [Template Sensors](https://www.home-assistant.io/integrations/template/)
  - Template sensors needed for the frontend. See [templates.yaml](templates.yaml).

<h3>Additional Sensors (with attributes) created by the Node-RED API calls</h3>

- **Total Daily Feeding *(since midnight)***
  - Daily bottles counts
  - Average per bottle
- **Total Daily Pumping Amount *(since midnight)***
  - Times pumped
  - Average pumping amount
- **Total Daily Diaper Changes *(since midnight)***
- **Bottle Counts Day *(8am-8pm)***
  - Total
  - Average per bottle
- **Bottle Counts Night *(8pm-8am)***
  - Total
  - Average per bottle
- **Daily Wet Diapers *(since midnight)***
- **Daily Solid Diapers *(since midnight)***
- **Rolling 24hr Feeding Avg**
  - Bottle count
  - Total amount in last 24hr
- **Rolling 24hr Diaper Avg**
- **Rolling 24hr Wet Diaper Avg**
- **Rolling 24hr Solid Diaper Avg**

Our newborn was a late preterm, so the additional information was useful to us. Recent studies have shown that daytime breastmilk has more cortisol and nighttime breastmilk has more melatonin. So my wife tries to split her breastmilk based on when it was pumped. This meant tracking bottles/averages of day and night separately to help us prevent waste when we fortify it.

<h2>Breakdown</h2>

<h3>Lovelace</h3>
My frontend changes include formatting changes, inclusion of graphs for each section, and the addition of the above data. Markdown headers now separate sections of data making it easier to read. The biggest change was modifying the `Start Timer` button, which now animates when the timer is active, and it also displays an elapsed time to gauge duration of the activity. The End Feeding/Sleep/TummyTime buttons are now hidden behind a conditional card that only shows when the timer is active. I retained the subviews for each activity, but I duplicated relevant information into their respective subviews as well. The only downside of the subviews is they don't navigate back to the previous lovelace page since only one action is allowed, which is needed for the helper.

#### Code: [lovelace.yaml](lovelace.yaml)

<h3>Node-RED Automations</h3>

I’m a very visual person, so I use Node-RED for 98% of my automations. The automations used in this project use a variety of inputs, including input helpers and Noblewolf's blueprint.

One of the things I set out to do with this project was to get more information from BabyBuddy into my frontend. I noticed that by strictly using input helpers to calculate things, my data would sometimes be mismatched. For example, if I added a feeding to BabyBuddy directly, my feeding counter helper wouldn't include it. To solve this, I decided to go directly to the source itself via the API. Using Node-RED, I paired the input helpers together with API calls to POST and GET all the data I wanted, including accurate counts for things.

Since these projects are intended for the sleep-deprived parents of newborns, mistakes are almost guaranteed. Failure to input all required information meant no data would POST to the add-on, resulting in missed entries. The Node-RED flows now check for what data is present vs what is required to prevent API errors. When required data is found to be missing, the API call is bypassed, the current time is stored as an `endTime` variable, and a mobile notification is sent to complete it at our earliest convenience. Using the `endTime` var allows us to finish whatever we are doing without worrying about times getting messed up. We also get confirmation notifications after the integration POSTs the information successfully. I added a persistent notification that can quickly tell us when our child ate last and how much, without having to open up the dashboard.

#### Code: [Node-RED.json](Node-Red.json)

<h2>Known issues:</h2>

- API calls on the first day of the month that reference the previous day *(ex. overnight bottle counts)* do not work properly based on how the function node builds the `url` variable.
- If the previous feeding/pumping amount is the same as the new amount, the `State` for the BabyBuddy HA Integration's `last_feeding` and `last_pumping` sensors don't update, only the attributes update. The fix is to use the template sensors. The template sensors extract a variety of these attributes which can then be used as triggers in Node-RED.
