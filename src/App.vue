<template>
  <div id="app">
    <nav-bar v-if='this.$route.path != "/"' />
    <router-view />  
  </div>
</template>

<script>
import {
  PushNotifications } from '@capacitor/push-notifications';
import NavBar from './components/NavBar.vue'
export default {
  name: 'App',
  components: {
    NavBar
  },
  data() {
    return {
      
    }
  },
  mounted() {
    PushNotifications.requestPermissions().then(result => {
      if (result.receive === 'granted') {
        // Register with Apple / Google to receive push via APNS/FCM
        PushNotifications.register();
      } else {
        console.log("Error")
      }
    });
    PushNotifications.addListener('registration',
      (token) => {
        alert('Push registration success: '+ token.value);
      }
    );
    PushNotifications.addListener('registrationError',
      (error) => {
        alert('Error on registration: ' + JSON.stringify(error));
      }
    );
    PushNotifications.addListener('pushNotificationReceived',
      (notification) => {
        alert('Push received: ' + JSON.stringify(notification));
      }
    );

    // Method called when tapping on a notification
    PushNotifications.addListener('pushNotificationActionPerformed',
      (notification) => {
        alert('Push action performed: ' + JSON.stringify(notification));
      }
    );
  }
};
</script>
