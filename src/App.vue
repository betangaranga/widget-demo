/* eslint-disable no-unused-vars */
<template>
  <div id="app">
    <div id="belvo" />
    <div id="finishText" v-if="processFinished">
      <p>LOS DATOS HAN COMENZADO A SER RECOPILADOS</p>
    </div>
  </div>
</template>
<style scoped>
  #finishText{
      color: green;
      font-size: large
  }
</style>
<script>
//import axios from "axios";
export default {
  name: 'App',
  data: ()=> ({
    processFinished: false,
    belvoAccessToken: null,
    chronosToken: null,
    chronosUserToken: null,
    chronosAuth: 'Basic N245cnJkcTlnZWhpdW85cG1sbWl1aTRsN3M6MWhmb3EzdGJmc2dqOGM3MWg0dWMzbXJhdW4yNHBnYjMyaXBtNmdyamU2amE1cWFoMmQ1aw==',
    chronosAuthUrl: 'https://api-ion-chrono-dev.auth.us-east-1.amazoncognito.com/oauth2/token',
    chronosUrl: 'https://5tenxo9kni.execute-api.us-east-1.amazonaws.com/dev/'
  }),
  async mounted () {
            await this.setChronosToken()

            await this.setUserToken()

            window.belvoSDK = require('belvo');

            this.loadScript('https://cdn.belvo.io/belvo-widget-1-stable.js');
      },
      methods: {
        
        async setUserToken(){
          const requestUrl = `${this.chronosUrl}authentication/sign-in`
          const headers = new Headers()
          headers.append('Authorization', this.chronosToken)
          const requestOptions = {
            method: 'PUT',
            headers: headers,
            body: JSON.stringify({
                "email": "00.supernov4.00+p39@gmail.com",
                "password": "test1234"
              })
          }
          const response = await fetch(requestUrl, requestOptions)
          const responseText = await response.text();
          const parsedResponse = JSON.parse(responseText)
          this.chronosUserToken = parsedResponse.data.auth_token
        },
        async setChronosToken(){
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
          await this.setUserToken()
          //await this.setChronosToken()
          const requestUrl = `${this.chronosUrl}bank_connection/init`
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
        async initDataFetching(link){
          const requestUrl = `${this.chronosUrl}bank_connection/`
          const headers = new Headers()
          headers.append('Authorization', this.chronosUserToken)
          const requestOptions = {
            method: 'POST',
            headers: headers,
            body: JSON.stringify({
                "external_provider_id": link
              })
          }
          const response = await fetch(requestUrl, requestOptions);
          const responseText = await response.text();
          console.log("Fetching Response: ", responseText)
   
        },
        async createWidget() {
        const successCallbackFunction = async(link, institution) => {
          await this.initDataFetching(link)
          console.log(link, institution)
          this.processFinished = true;
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


