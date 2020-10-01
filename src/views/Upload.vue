<template><div>
    <p>You need to be logged in to upload a video</p>
    <input type = 'file' accept = 'video/*' @change = 'onFileChange' />
    <button @click = 'uploadVideo' :disabled = '!video'>Upload Video</button>
</div></template>

<script>
import nearConfig from '../nearConfig.js';
import * as nearApi from 'near-api-js';
import { SkynetClient } from 'skynet-js';
export default {
    data() {
        return {
            video: null
        }
    },
    methods: {
        onFileChange(e) {
            let files = e.target.files || e.dataTransfer.files;
            this.video = files[0];
            console.log('FILES', files);
            if(!files.length) return;
        },
        uploadVideo: function() {
            const video = this.video;
            const client = new SkynetClient('https://siasky.net');
            return client.uploadFile(video)
            .then(skylink => {
                console.log('SKYLINK', skylink);
            }).catch(err => {
                console.log('ERR UPLOADING VIDEO TO SKYNET', err);
            });
        }
    },
    mounted() {
        console.log('MOUNTED');
        let walletConnection;
        return nearApi.connect({ ...nearConfig('development'),  deps: {
            keyStore: new nearApi.keyStores.BrowserLocalStorageKeyStore() // keys are stored as plaintext in LocalStorage
        }})
        .then(res => {
            window.near = res;
            // window.wallet = new nearApi.WalletAccount(res)
            walletConnection = new nearApi.WalletConnection(res);

            return walletConnection.account().state()
        })
        .then(balance => {
            let currentUser
            if(walletConnection.getAccountId()) {
                currentUser = {
                    accountId: walletConnection().getAccountId(),
                    balance: balance.amount
                }
                console.log('CURRENT USER', currentUser);
            }

        });
    }
}
</script>
