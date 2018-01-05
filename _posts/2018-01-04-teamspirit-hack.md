---
layout: post
title: "TeamSpirit Hack"
description: "Hack to internal structure of TeamSpirit Salesforce objects to integrate with other applications"
tags: [web, note]
---
## Purpose
TeamSpirit is an application running on Salesforce to provide employee management in many aspects. However, users were only able to do the 打刻(time registration) operations from the web application website. TeamSpirit currently refuse to provide open APIs for developers to access their application from the 3rd party. Here we want to export the interface to our chatbot to do time registration automatically (and also possible with webcam integration, depends how you want to match users with the salesforce ids).

## Solution
To achieve our goal, we need to have the API priviledge of Salesforce. Just keep in mind that all applications running on Salesforce stores user data in the database of that company. To access Salesforce API, we need to have the following constants:

1. salesforce_base: the domain of your salesforce service. ex: https://abc.my.salesforce.com
2. user: user email
3. password: login password
4. token: API token
5. client_id: client API ID
6. client_secret: client secret code

### Salesforce Authentication
Here will going to show the authentication process in golang. We will create an auth agent instance to hold the access token and auto-refresh it after 3 hours (This depends on the admin settings of valid authentication period, normally it's more than 3 hours).

   ```go
   // define a salesforce client
   type Agent struct {
       AuthTime time.Time
       Key string
   }

   // for getting new instance
   func NewAgent() *Agent {
       return &Agent {
           AuthTime: time.Now(),
       }
   }
   ```

And as for authentication, we use oauth2:

   ```go
   // for json unmarshal
   type AuthType struct {
     Access_token string `json:"access_token"`
   }
   // keep it private. we wish to call it lazily on operations
   func (a *Agent) auth() error {
       // if we already have key and is still valid
       if len(a.Key) > 0 &&
           time.Now().Sub(a.AuthTime) < time.Duration(3)*time.Hour {
           return nil
       }
       resp, err := http.PostForm(
           salesforce_base + "/services/oauth2/token",
           url.Values{
               "grant_type": {"password"},
               "client_id": {client_id},
               "client_secret": {client_secret},
               "username": {user},
               "password": {password+token},
           },
       )
       if err != nil {
         return err
       }
       defer resp.Body.Close()
       body, err := ioutil.ReadAll(resp.Body)
       if err != nil {
           return err
       }
       if resp.StatusCode != 200 {
           return fmt.Errorf("Auth fail: %d, %s\n", resp.StatusCode, string(body))
       }
       var authObj AuthType
       if err = json.Unmarshal(body, &authObj); err != nil {
           return err
       }
       a.Key = authObj.Access_token
       a.AuthTime = time.Now()
       return nil
   }
   ```

Now we could create a test file for testing. Given the implementation name `salesforce.go`, the test file should be `salesforce_test.go`.

   ```go
   func TestLogin(t *testing.T) {
       a := NewAgent()
       if err := a.auth(); err != nil {
           t.Error(err)
           return
       }
   }
   ```

If everything goes fine, let's go back to the TeamSpirit definition. Login to your Salesforce console and go to `Setup -> Build -> Develop -> Installed Packages`. You could find an app named TeamSpirit over that page:
{% include image.html path="20180104/installed.png" path-detail="20180104/installed.png" alt="Installed apps"%}

And click on the `View Components` button to see all TeamSpirit tables:
{% include image.html path="20180104/tables.png" path-detail="20180104/tables.png"%}

Here are all the tables that created by TeamSpirit. Click into any table and you could see the definition of each column and table details:
{% include image.html path="20180104/object.png" path-detail="20180104/object.png"%}
{% include image.html path="20180104/column.png" path-detail="20180104/column.png"%}


To query out the data, we could either use developer console together with SOQL, or use query API provided by Salesforce. Here's the sample query function:
   ```go
   // query result structure
   type QueryStruct struct {
       TotalSize      int                      `json:"totalSize"`
       Records        []map[string]interface{} `json:"records"`
       NextRecordsUrl string                   `json:"nextRecordUrl"`
       Done           bool                     `json:"done"`
   }
   // query with soql string
   func (a *Agent) Query(soql string) (*QueryStruct, error) {
       if err := a.auth(); err != nil {
           return nil, err
       }
       query := (url.Values{"q": {soql}}).Encode()
       rq, err := http.NewRequest(
           "GET",
           salesforce_base+"/services/data/v32.0/query/?"+query,
           nil,
       )
       if err != nil {
           return nil, err
       }
       rq.Header = map[string][]string{
           "Authorization": {"Bearer " + a.Key}
       }
       r, err := http.DefaultClient.Do(rq)
       if err != nil {
         return nil, err
       }
       defer r.Body.Close()
       var qs QueryStruct
       if r.StatusCode == 200 || r.StatusCode == 204 {
           dec := json.NewDecoder(r.Body)
           err = dec.Decode(&qs)
           if err != nil {
               return nil, err
           }
           return &qs, nil
       }
       return nil, fmt.Errorf("fail on StatusCode: %v", r.Status)
   }
   ```

Here's the most important table (teamspirit__AtkEmpDay__c) for time registration:

| Column Name                           | Definition                    | Example            |
|---------------------------------------|-------------------------------|--------------------|
| Id                                    | row index                     |                    |
| Name                                  | row name, user name + date    | ABC2017-10-03      |
| CreatedById                           | TeamSpirit user id            | 00510000005YTuvAAG |
| teamspirit__PushStartTime__c          | time registration start time  | 540                |
| teamspirit__StartTime__c              | start work time               | 540                |
| teamspirit__RealStartTime__c          | start work time               | 540                |
| teamspirit__EndTime__c                | end work time                 | 1080               |
| teamspirit__RealEndTime__c            | end work time                 | 1080               |
| teamspirit__PushEndTime__c            | time registration end time    | 1080               |
| teamspirit__WorkOffTime__c            | rest time                     | 60                 |
| teamspirit__restTime__c               | regulated rest time           | 60                 |
| teamspirit__WorkFixedTime__c          | regulated work time           | 480                |
| teamspirit__WorkLegalTime__c          | legal work time               | 480                |
| teamspirit__WeekDayDayLegalFixTime__c | legal work time in weekday    | 480                |
| teamspirit__WeekDayDayLegalOutTime__c | exceeded work time in weekday | 10                 |
| teamspirit__WorkRealTime__c           | actual work time              | 490                |
| teamspirit__WorkWholeTime__c          | total work time               | 490                |
| teamspirit__WorkNetTime__c            | net work time                 | 490                |
| teamspirit__WorkOverTime__c           | over work time                | 10                 |
| teamspirit__WorkLegalOutOverTime__c   | legal over work time          | 10                 |
| teamspirit__TimeTable__c              | time distribution             | 0720078021:        |
| teamspirit__DayType__c                | day type, 0=working day, others for rest | 1       |


Many other columns are not included because most of them are only references and formulas. We only have to update the necessary columns to change the view.

Other useful tables might be useful in otherways. For example, to get the Id mapping, we could go to teamspirit__AtkEmp__c for Salesforce ID to TeamSpirit ID, and onboard date of that user. As for email and Salesforce ID mapping, get from User table.

TBD
