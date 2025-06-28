<script setup lang="ts">
import { Barcode, BarcodeScanner } from '@capacitor-mlkit/barcode-scanning';
import { scan } from 'ionicons/icons';
import { onMounted, ref } from 'vue';
import {
  IonPage,
  IonContent,
  IonHeader,
  IonToolbar,
  IonTitle,
  alertController,
  IonFooter,
  IonFabButton,
  IonIcon,
  IonList,
  IonItem,
  IonGrid,
  IonRow,
  IonCol,
  IonNote,
  IonInput,
  IonLabel,
} from '@ionic/vue';

const isSupported = ref(false);
const barcodeList = ref([] as Barcode[]);

onMounted(async () => {
  isSupported.value = (await BarcodeScanner.isSupported()).supported;
});

const startScanning = async () => {
  const granted = await requestPermissions();

  if (!granted) {
    presentAlert(
      'Permission denied',
      'Please grant camera permission to use the barcode scanner.'
    );
    return;
  }

  const { barcodes } = await BarcodeScanner.scan();
  barcodeList.value.push(...barcodes);
};

const requestPermissions = async () => {
  const { camera } = await BarcodeScanner.requestPermissions();
  return camera === 'granted' || camera === 'limited';
};

const presentAlert = async (hdr: string, msg: string) => {
  const alert = await alertController.create({
    header: hdr,
    message: msg,
    buttons: ['OK'],
  });
  await alert.present();
};
</script>
<template>
  <ion-page>
    <ion-header>
      <ion-toolbar
        ><ion-title>NadetDev Barcode Scanner</ion-title></ion-toolbar
      ></ion-header
    >
    <ion-content>
      <ion-list v-if="barcodeList.length > 0">
        <ion-item v-for="barcode in barcodeList" :key="barcode.rawValue">
          <io-grid>
            <ion-row>
              <ion-col>
                <ion-input
                  type="text"
                  :value="barcode.rawValue"
                  :readonly="true"
                ></ion-input> </ion-col
            ></ion-row>
            <ion-row class="ion-text-center">
              <ion-col
                ><ion-label>Format</ion-label
                ><ion-note>{{ barcode.format }}</ion-note></ion-col
              ><ion-col>
                <ion-label>Type</ion-label>
                <ion-note>{{ barcode.valueType }}</ion-note>
              </ion-col>
            </ion-row>
          </io-grid>
        </ion-item>
      </ion-list>
    </ion-content>
    <ion-footer class="ion-padding">
      <ion-fab-button
        style="margin: auto"
        v-if="isSupported"
        @click="startScanning()"
      >
        <ion-icon :icon="scan"></ion-icon>
      </ion-fab-button>
    </ion-footer>
  </ion-page>
</template>
