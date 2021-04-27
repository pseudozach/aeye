<template>
    <div style="display: inline; margin-right: 5px;" v-if="userData == null">
        <VueLoadingButton 
            aria-label="Sign in"
            class="button"
            @click.native="authenticate"
            :loading="isLoading"
            :styled="true"
        >Sign in
        </VueLoadingButton>
    </div>
</template>

<style>
    
</style>

<script>
    // import { Box, Text, Button } from '@blockstack/ui';
    import VueLoadingButton from 'vue-loading-button'
    import { AppConfig, UserSession, showConnect } from '@stacks/connect';
    import { Person } from '@stacks/profile';

    const appConfig = new AppConfig(['store_write', 'publish_data']);
    const userSession = new UserSession({ appConfig });

    export default {
        components: {
            VueLoadingButton
        },
        data(){
            return {
                msg: "Siging in...",
                isLoading: false,
                userData: null,
            }
        },
        mounted: function () {
            // console.log("mounted ", userSession, userSession.loadUserData());
            this.userData = userSession.loadUserData();
        },
        methods:{
            onComplete(data){
                console.log('data:', data);
            },
            authenticate(){
                console.log("authenticate");
                var thisthing = this;
                  if (userSession.isSignInPending()) {
                    console.log("isSignInPending");
                    userSession.handlePendingSignIn().then((userData) => {
                      window.history.replaceState({}, document.title, "/")
                      thisthing.userData = userData;
                    });
                  } else if (userSession.isUserSignedIn()) {
                    console.log("isUserSignedIn");
                    this.userData = userSession.loadUserData();
                  } else {
                    console.log("showConnect");
                    showConnect({
                        appDetails: {
                          name: 'Stxpredict',
                          icon: window.location.origin + '/icon.png',
                        },
                        redirectTo: '/',
                        finished: () => {
                          window.location.reload();
                        },
                        userSession: userSession,
                    });  
                  }

            }
        }
    }
</script>