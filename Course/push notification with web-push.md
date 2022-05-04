# push notification with web push

### service-worker.js

```js
self.addEventListener('push', function (e) {
    e.waitUntil((async () => {
        const data = e.data.json();

        let options = {
            body: data.message,
            icon: '/img/icons/favicon.png',        
        };
        await self.registration.showNotification(data.title, options);
    })());
});
```

### vue.config.js

```js
pwa: {
    name: 'projectName',
    themeColor: '#006064',
    msTileColor: '#000000',
    appleMobileWebAppCapable: 'yes',
    appleMobileWebAppStatusBarStyle: 'black',

    // configure the workbox plugin
    workboxPluginMode: 'InjectManifest',
    workboxOptions: {
      // swSrc is required in InjectManifest mode.
      swSrc: 'src/service-worker.js',
      // ...other Workbox options...
    }
  }
```

### PushNotificationHelper.js

```js
import store from '../store'
export default {
    /**
     * Register the service worker.
     */
    registerServiceWorker() {
        console.log('registerServiceWorker');
        if (!('serviceWorker' in navigator)) {
            console.log('Service workers aren\'t supported in this browser.')
            return
        }
        navigator.serviceWorker.register('service-worker.js')
            .then(() => this.init())
    },

    async init() {
        this.findSubscription().then(sub => {
            if (sub === null) {
                console.log('no active subscription found on the client', sub);

                // Ask permission and when granted, create new subscription
                Notification.requestPermission()
                    .then(result => {
                        console.log(result);
                        // if granted, create new subscription
                        if (result === 'granted') {
                            this.createSubscription()
                            
                        } else {
                            console.log('User did not granted permission')
                        }
                    })
            } else {
                console.log('Active subscription found', JSON.stringify(sub));
                // retrieve user info from API
                
                this.addSubscription(sub)
            }
        })
    },
    findSubscription() {
        console.log('get active service worker registration');
        return navigator.serviceWorker.ready
            .then(swreg => {
                console.log('haal active subscription op');
                this.serviceWorkerRegistation = swreg
                return this.getSubscription(this.serviceWorkerRegistation)
            })
    },
    getSubscription(swreg) {
        console.log('ask for available subscription');
        return swreg.pushManager.getSubscription()
    },
    async createSubscription() {
        let sw = await navigator.serviceWorker.ready
        let subscription = await this.subscribe(sw)
        this.addSubscription(subscription)
    },
    addSubscription(subscription) {
        
        var p256dh = this.base64Encode(subscription.getKey('p256dh'));
        var auth = this.base64Encode(subscription.getKey('auth'));

        const data = {
            endpoint: subscription.endpoint,
            p256dh: p256dh,
            auth: auth 
        }

        store.dispatch('NotificationAddDevice', data).then(result => {
            console.log(result);
        })
    },

    base64Encode(arrayBuffer) {
        return btoa(String.fromCharCode.apply(null, new Uint8Array(arrayBuffer)));
    },
    urlB64ToUint8Array(base64String) {
        const padding = '='.repeat((4 - base64String.length % 4) % 4);
        const base64 = (base64String + padding)
            .replace(/-/g, '+')
            .replace(/_/g, '/');

        const rawData = window.atob(base64);
        const outputArray = new Uint8Array(rawData.length);

        for (let i = 0; i < rawData.length; ++i) {
            outputArray[i] = rawData.charCodeAt(i);
        }
        return outputArray;
    },

    async subscribe(swreg) {
        console.log('create new subscription for this browser on this device');
        // create new subscription for this browser on this device
        const vapidPublicKey = process.env.VUE_APP_VAPID_PUBLIC_KEY
        console.log(vapidPublicKey);
        const convertedVapidPublicKey = this.urlB64ToUint8Array(vapidPublicKey)

        let push = await swreg.pushManager.subscribe({
            userVisibleOnly: true,
            // This is for security. On the backend, we need to do something with the VAPID_PRIVATE_KEY
            // that you can find in .env to make this work in the end
            applicationServerKey: convertedVapidPublicKey
        })
        // return the subscription promise, we chain another then where we can send it to the server
        console.log(JSON.stringify(push));
        return push
    },

    showGrantNotification() {
        this.serviceWorkerRegistation.showNotification('Notifications granted', {
            body: 'Here is a first notification',
            icon: '/img/icons/android-chrome-192x192.png',
            image: '/img/autumn-forest.png',
            vibrate: [300, 200, 300],
            badge: '/img/icons/plint-badge-96x96.png',
        })
    },

}
```

***

<a href="http://www.aliakbar-ghasemi.ir" style="display:flex">
    <img src="https://avatars.githubusercontent.com/u/33986952?s=96&v=4" alt="aliakbar-ghasemi.ir" width="50"
        style="border-radius: 50%;padding:10px" />
    <div style="display: flex;
    align-items: center;">
        <div style="display:flex; flex-direction: column;">
            <span>علی اکبر قاسمی</span>
            <span>http://www.aliakbar-ghasemi.ir</span>
        </div>
    </div>
</a>