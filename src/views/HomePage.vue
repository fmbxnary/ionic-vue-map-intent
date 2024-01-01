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
      <ion-button @click="openSelectedMap">Open Map</ion-button>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonItem, IonLabel, IonInput, IonSelect, IonSelectOption, IonButton, toastController } from '@ionic/vue';
import { Capacitor } from '@capacitor/core';
import { AppLauncher } from '@capacitor/app-launcher';
import { ref, computed } from 'vue';

const lat = ref('');
const lng = ref('');
const selectedMap = ref('google');

const packageName = computed(() => {
  const platform = Capacitor.getPlatform();
  if (platform === 'android') {
    return selectedMap.value === 'google' ? 'com.google.android.apps.maps' : 'ru.yandex.yandexmaps';
  } else if (platform === 'ios') {
    return selectedMap.value === 'google' ? 'comgooglemaps://' : 'yandexmaps://';
  }
  return '';
});

const createToast = async (message) => {
  const toast = await toastController.create({
    message: message,
    duration: 2000,
    color: 'danger',
  });
  toast.present();
};

const openSelectedMap = async () => {
  if (!lat.value || !lng.value) {
    createToast('Please fill in all fields');
    return;
  }
  try {
    const result = await AppLauncher.canOpenUrl({ url: packageName.value });
    if (result) {
      const url = selectedMap.value === 'google'
        ? `geo:${lat.value},${lng.value}?z=14&q=${lat.value},${lng.value}`
        : `yandexmaps://maps.yandex.ru/?pt=${lat.value},${lng.value}&z=14`;
      await AppLauncher.openUrl({ url: url });
    } else {
      createToast(`${selectedMap.value === 'google' ? 'Google Maps' : 'Yandex Maps'} is not installed on this device`);
    }
  } catch (error) {
    console.error(error);
    createToast('An error occurred while opening the map');
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
