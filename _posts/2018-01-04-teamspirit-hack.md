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

TBD.
