<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>
          Map App
        </ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <ion-item>
        <ion-label position="floating">Latitude</ion-label>
        <ion-input v-model="lat"></ion-input>
      </ion-item>
      <ion-item>
        <ion-label position="floating">Longitude</ion-label>
        <ion-input v-model="lng"></ion-input>
      </ion-item>
      <ion-item>
        <ion-label>Map</ion-label>
        <ion-select v-model="selectedMap" interface="popover">
          <ion-select-option value="google">Google Maps</ion-select-option>
          <ion-select-option value="yandex">Yandex Maps</ion-select-option>
        </ion-select>
      </ion-item>
      <ion-button @click="openSelectedMap(selectedMap)">Open Map</ion-button>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar } from '@ionic/vue';
import { IonItem, IonLabel, IonInput, IonSelect, IonSelectOption, IonButton, toastController } from '@ionic/vue';
import { Capacitor } from '@capacitor/core';
import { AppLauncher } from '@capacitor/app-launcher';
import { ref } from 'vue';

const lat = ref('');
const lng = ref('');
const selectedMap = ref('');
/* create toast message for if lon or lng is empty */
const createToast = async (message) => {
  const toast = await toastController.create({
    message: message,
    duration: 2000,
    color: 'danger',
  });
  toast.present();
};

const openSelectedMap = async (mapApp) => {
  /* if lat or lng empty alert ionic */
  if (!lat.value || !lng.value || !selectedMap.value) {
    createToast('Please fill in all fields');
    return;
  }
  try {
    let packageName;
    if (Capacitor.getPlatform() === 'android') {
      packageName = mapApp === 'google' ? 'com.google.android.apps.maps' : 'ru.yandex.yandexmaps';
    } else if (Capacitor.getPlatform() === 'ios') {
      packageName = mapApp === 'google' ? 'comgooglemaps://' : 'yandexmaps://';
    }
    const result = await AppLauncher.canOpenUrl({ url: packageName });
    if (result) {
      let url;
      if (mapApp === 'google') {
        url = `geo:${lat.value},${lng.value}?z=14&q=${lat.value},${lng.value}`;
      } else {
        url = `yandexmaps://maps.yandex.ru/?pt=${lat.value},${lng.value}&z=14`;
      }
      await AppLauncher.openUrl({ url: url });
    } else {
      console.log(`${mapApp === 'google' ? 'Google Maps' : 'Yandex Maps'} is not installed on this device`);
    }
  } catch (error) {
    console.log(error);
  }
};

</script>

<style scoped>
#container {
  text-align: center;

  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;

  color: #8c8c8c;

  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
