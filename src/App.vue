/* eslint-disable no-unused-vars */
<template>
  <div id="app">
    <div id="belvo" />
  </div>
</template>

<script>
//import axios from "axios";
export default {
  name: 'App',
  data: ()=> ({
    belvoAccessToken: null,
    chronosToken: null,
    chronosAuth: 'Basic N245cnJkcTlnZWhpdW85cG1sbWl1aTRsN3M6MWhmb3EzdGJmc2dqOGM3MWg0dWMzbXJhdW4yNHBnYjMyaXBtNmdyamU2amE1cWFoMmQ1aw==',
    chronosAuthUrl: 'https://api-ion-chrono-dev.auth.us-east-1.amazoncognito.com/oauth2/token',
    chronosUrl: 'https://5tenxo9kni.execute-api.us-east-1.amazonaws.com/dev/'
  }),
  created(){
    /*axios({
      method: 'post',
      url: this.chronosAuthUrl,
      data: qs.stringify({
        grant_type: 'client_credentials',
        scope: 'leads/simulation-read leads/lead-create verification/verification-read verification/verification-create'
      }),
      headers: {
          'Authorization': this.chronosAuth,
          'Content-Type': 'application/x-www-form-urlencoded;charset=utf-8'
        }
    })
      .then((result)=> {
          this.chronosToken = result.data.access_token
          console.debug("Access token: ", this.chronosToken)
        })
        .catch((err) => {
          console.error("Error getting acces token: ", err, this.chronosAuthUrl)

    // Do somthing
        })*/
          //const requestUrl = `${this.chronosAuthUrl}/${process.env.SHAREPOINT_SITE}`

  },
  mounted () {
    window.belvoSDK = require('belvo');

        // Executing the function to load the widget after mount, since we need
        // the template to be loaded
        this.loadScript('https://cdn.belvo.io/belvo-widget-1-stable.js');
      },
      methods: {
        async getChronosToken(){
          const headers = new Headers()
          headers.append('Authorization', this.chronosAuth)
          headers.append('Content-Type', 'application/x-www-form-urlencoded')
          const requestOptions = {
            method: 'POST',
            headers: headers,
            body: new URLSearchParams({
                'grant_type': 'client_credentials',
                'scope': 'leads/simulation-read leads/lead-create verification/verification-read verification/verification-create',
            })
          }
          const response = await fetch(this.chronosAuthUrl, requestOptions)
          const responseText = await response.text();
          const parsedResponse = JSON.parse(responseText)
          this.chronosToken = parsedResponse.access_token;
        },
        async getBelvoAccessToken() {
          await this.getChronosToken()
          const requestUrl = `${this.chronosUrl}/bank_connection/init`
          const headers = new Headers()
          headers.append('Authorization', this.chronosToken)
          const requestOptions = {
            method: 'POST',
            headers: headers,
            redirect: 'follow'
          }
          const response = await fetch(requestUrl, requestOptions)
          const responseText = await response.text();
          const parsedResponse = JSON.parse(responseText)
          return parsedResponse.data.access_token
          
        },
        async createWidget() {
        const successCallbackFunction = (link, institution) => {
          console.log(link, institution)
            // Do something with the link and institution,
            // such as associate it with your registered user in your database.
        }
        const onExitCallbackFunction = (data) => {
                    console.log(data)

            // Do something with the exit data.
        }
        const onEventCallbackFunction = (data) => {
          console.log(data)
            // Do something with the exit data.
        }

        const config = {

            // Add your startup configuration here.

            callback: (link, institution) => successCallbackFunction(link, institution),
            onExit: (data) => onExitCallbackFunction(data),
            onEvent: (data) => onEventCallbackFunction(data)
        }
        //setAccessToken(this.chronosUrl, this.chronosToken, this)
        await this.getChronosToken()
        this.belvoAccessToken =  await this.getBelvoAccessToken()
        window.belvoSDK.createWidget(this.belvoAccessToken, config).build()
    },
        loadScript(src) {
        const node = document.createElement('script');
        node.src = src;
        node.type = 'text/javascript';
        node.async = true;
        // Assign the callback which will create the Widget
        node.onload = this.createWidget;
        console.debug("Script: ", node)
        // Add script to document body
        document.body.appendChild(node);
    }
        
        // Please add the code from Step 3 here.
      }
  
}
</script>


