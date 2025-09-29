***Problem Statement*** 

* **Domain \- Taking ownership of health:** With healthcare systems often difficult to access and 

medical misinformation on the rise, it’s increasingly important for people to understand their own bodies and have intuitive, user-friendly tools to take charge of their own health in a safe way. I want to empower people to track aspects of their daily physical health and present the resulting information in meaningful ways to identify patterns and have clear, data-informed conversations with healthcare professionals.

* **Problem \- Communicating the prolonged nature and severity of chronic pain:** Chronic pain is a 

complex and frequently invisible condition that affects millions of people but the subjective nature of pain makes it difficult to evaluate. There is also difficulty in presenting consistent data over time, preventing medical professionals from validating the progression of the condition. I believe that there is a need for a low-effort tool to track and communicate chronic pain, considering that patients are frequently denied appropriate care because they are unable to accurately articulate what they are regularly enduring.

* **Stakeholders:** Patient (the user who experiences chronic pain and interacts with the app); Inner

Circle (someone personally close to the user who hears about their condition regularly); Healthcare Professionals (someone who will review the data collected and analysed by the app); Insurance Companies (someone who may have to cover more treatment); Advocacy Groups (someone who uses the app to support their cause for awareness). Impact: Both Patient and Healthcare Professionals suffer from poorly presented data, but Patient suffers from lack of validation. Inner Circle presumably cares about Patient’s well-being so they will benefit from Patient’s healthcare needs being met. Insurance Companies will be less than thrilled about the app's success as it would mean more coverage for some current customers, but they could also gain new customers. 

* **Citations  \+ Comparables**

[’I was told to live with it’: women tell of doctors dismissing their pain](https://www.theguardian.com/society/2021/apr/16/painkillers-women-tell-of-doctors-dismissing-their-pain): An article about  women dealing with years-long pain that drastically affected their quality-of-life and still not being believed by doctors.  
[Women and pain: Disparities in experience and treatment](https://www.health.harvard.edu/blog/women-and-pain-disparities-in-experience-and-treatment-2017100912562): An article about women tending to experience more stigma and discrimination on average when seeking treatment for chronic pain.   
[When Nobody Believes You](https://www.jennifermartinpsych.com/yourcolorlooksgoodblog/2015/8/3/when-nobody-believes-you): A personal account of a psychologist who discusses her patients’ experiences with dismissals of chronic pain as ‘fake’ or purely psychological issues to overcome.  
[Bearable](https://bearable.app): A symptom tracker app for health issues and general habits related to well-being that analyses and identifies trends based on personal accounts, such as how caffeine consumption levels have affected your sleep quality.  
[Health Storylines](https://www.healthstorylines.com): An app that stores and tracks medical information such as medications, weight, blood pressure, and more to provide a clear summary when visiting a healthcare professional. It also has a journal feature that allows users to log moods and daily states.

***Application Pitch***

* **Name: PainPal**

* **Motivation:** PainPal helps people with chronic pain effectively track and communicate their daily experiences and long-term symptoms, transforming it into clear, credible data that can be shared with healthcare providers for a more validating response and appropriate treatment.

* **Key features**  
1. BodyMap: Users are presented with a 360-degree silhouette of a human body on which they can highlight areas that are hurting on a given day and rate each the pain of each highlighted region on a scale of 1-10. This is a way for users to record their symptoms on a daily basis (or even update the Map multiple times a day) which can then be summarised for presentation to a healthcare provider. 

2. BreakThrough Log: (Context: Breakthrough pains are a sudden, temporary, and often severe increase in pain.) Users can log when they experience breakthrough pains or intense flare-ups, how long they last for, and select commonly used medical terms to describe them. These will then be converted into statistical summaries that inform a healthcare provider about how often a patient is experiencing intense pain. 

***Concept Design***

* **Concept Specifications**

**(1)**  
**concept** BodyMapGeneration  
**purpose** a body map is saved at the end of each day and a fresh one is created for usage the next day  
**principle**   
**state**   
a set of Users with  
   a set of calendar Days

a set of calendar Days with  
   a set of body Maps

a set of body Maps with  
   a set of Highlights

**actions**   
	**system** generateMap(day: Day, hours: 24): (bodymap: Map)  
	   **effects** creates and returns a fresh Map associated with that day  
	**system** saveMap(bodymap: Map, hours: 24):  
	   **requires** the Map must exist   
	   **effects** saves the Map associated with that day  
	clearMap(bodymap: Map): (bodymap: Map)  
	   **requires** the Map must exist and have at least one Highlight  
	   **effects** removes all Highlights from the Map and returns it  
	   

**(2)**  
**concept** PainLocationScoring  
**purpose** users can select a region on the body map and rate the pain on a scale of 1 to 10  
**principle**   
**state**   
a set of Users with  
   a set of body Maps

a set of body Maps with  
   a set of Regions

a set of Regions with  
   a scaled score Number

**actions**   
	addRegion(map: Map, region: Region): (region: Region)  
	   **requires** the Map must already exist  
	   **effects** creates and returns a new Region on that Map  
	scoreRegion(region: Region, score: Number)  
   **requires**  the Region must exist and the Number must be between 1 and 10  
   **effects** associates the Number with that region  
deleteRegion(region: Region)  
   **requires** the Region must already exist  
   **effects** removes the Region from the associated Map

**(3)**  
**concept** MapSummaryGeneration  
**purpose** concisely summarise all body map logs until present-day  
**principle**   
**state**   
a set of Users with  
   a set of body Maps

a set of body Maps with  
   a set of Regions  
   a date Range

	a set of Regions with  
	   a frequency Number  
	   a median score Number  
	   a summary String     
**actions**   
	sumRegion(period: Range, mapSet: Maps, region: Region): (score: Number, frequency: Number)  
	   **requires** the Region must exist  
	   **effects** scans Maps in date Range, counts Region hits, returns Region with associated Numbers  
	summarise(period: Range, region: Region, score: Number, frequency: Number): (summary: String)  
   **requires** the Region and its Numbers must exist  
   **effects** returns a String incorporating the given values of Range, Region, and the Numbers

**(4)**  
**concept** BreakThroughTracking  
**purpose** record the occurrence and duration of breakthrough pain and summarise such occurrences  
**principle**   
**state**   
a set of Users with  
   a set of Months

a set of Months with  
   a set of Breakthroughs  
   a breakthrough Summary

a set of Breakthroughs with  
   a Start time  
   an End time

a breakthrough Summary with  
   a Number of breakthroughs  
   an average duration Number

**actions**   
	startBreakthrough(startTime: Start, month: Month): (pain: Breakthrough)  
	   **effects** creates and returns a new Breakthrough pain event associated with the Month  
	endBreakthrough(endTime: End, pain: Breakthrough): (pain: Breakthrough)  
	   **requires** the Breakthrough has a start time  
	   **effects** returns the Breakthrough with an End time to complete the log  
	summarise(month: Month, avgDuration: Number, frequency: Number) (summary: String)  
	   **effects**  returns a summary String with the frequency and average duration for the Month

* **Synchronisations**

**(1)**  
**sync** generateMap  
**when** BodyMapGeneration.saveMap(bodyMap: Map, hours: 24\)  
**then** BodyMapGeneration.generateMap(day: Day, hours: 24): (bodymap: Map)

**(2)**  
**sync** summarise  
**when** Request.displaySummary(summary: String)  
**then**   
   MapSummaryGeneration.sumRegion(period: Range, mapSet: Maps, region: Region): (score: Number, frequency: Number)  
   MapSummaryGeneration.summarise(period: Range, region: Region, score: Number, frequency: Number): (summary: String)

**(3)**  
**sync** deleteRegion  
**when** BodyMapGeneration.clearMap(bodymap: Map): (bodymap: Map)  
**then** PainLocationScoring.deleteRegion(region: Region)

* **Brief Note:** The PainPal app’s baseline concepts integrate to form features for chronic pain 

tracking. BodyMapGeneration manages per-user daily maps, as they are logged or saved every 24 hours, taking the pressure of remembering to save it off of the user’s shoulders and enabling organised data capturing. PainLocationScoring and MapSummaryGeneration forms the core functionality of the app as one operates on these maps by allowing users to define and score pain regions while the other aggregates this data to provide objective insight and highlight the consistency of an experience. BreakThroughTracking records and summarizes episodic breakthrough pains on a per-user monthly basis. Together, these concepts modularly handle user-friendly data logging, summarisation, and special event tracking, supporting PainPal’s goal of converting subjective pain reports into validated data for healthcare interactions.

***UI Sketches***

\!\[Sketch 1: Main Features\] (assignments/sketches/sketch1\_features.png)  
\!\[Sketch 2: Expanded Log Bar\] (assignments/sketches/sketch2\_logBar.png)

***User Journey***

A user wakes up with a noticeable level of pain for the sixth day in a row. This time, it extends from their right knee to their foot. They want to note it down for record purposes but find it tedious to manually keep track of all the details for more than 2-3 days. The user opens PainPal on their phone where a fresh BodyMap has been automatically generated since it is a new day. They press the right knee on the map and drag down to the ankle, leaving that area highlighted in orange. They tap on the newly highlighted area and type the number 7 into the empty field in the pop-up dialog that appears. 

They wonder if they are just imagining that this pain has been persistent for a while now. To confirm, they click on the summary button also contained in the pop-up for that region and select the start and end date of the past week from a drop-down calendar. A text summary is generated that highlights the statistics per region. For example, they would see the text ‘Over the last 7 days, you reported experiencing pain in your lower left leg area 6 times with a moderate median pain score of 6.5’. The user feels validated that it is not in their head. The user screenshots this summary or saves it to their camera rolland books an appointment with their primary physician as they can now support their claims with evidence (by showing them the screenshot of the summary, for example).