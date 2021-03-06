<template>
    <div class="flex flex-1 flex-col lg:p-10 sm:p-5 h-full">
        <div class="flex flex-col w-full lg:w-9/12 xl:w-8/12 mx-auto">
            <h1 class="text-2xl">Profile</h1>
            <div class="bg-sidebar-item p-5 rounded my-4">
                <div class="flex flex-col my-2 items-center">
                    <img :src="`https://minotar.net/helm/${avatarName}`" class="rounded-full" />
                    <p class="text-lg mt-2">{{auth.token.mc !== undefined ? auth.token.mc.display : auth.token.username}}</p>
                    <p>{{auth.token.activePlan !== undefined && auth.token.activePlan !== null ? auth.token.activePlan.name : ''}}</p>

                    <div class="mt-6 w-1/2">  
                        <ftb-button color="primary" class="text-center px-2 py-1" title="Coming Soon"  @click="openSubscriptions">Upgrade / Modify Subscription</ftb-button>
                        <ftb-button color="primary" class="text-center px-2 py-1 my-2" title="Coming Soon"  @click="openProfile">Update Profile</ftb-button>
                    </div>
                    <div class="mt-6 w-1/2">
                        <ftb-toggle label="Automatically open friends list" :value="settings.settings.autoOpenChat === true || settings.settings.autoOpenChat ==='true'"
                                onColor="bg-primary" inline="true" @change="toggleAutoOpenChat"/>
                        <ftb-toggle label="Enable cloud save uploads " :value="settings.settings.cloudSaves === true || settings.settings.cloudSaves ==='true'"
                                @change="toggleCloudSaves" :disabled="auth.token.activePlan === null"
                        onColor="bg-primary" inline="true"/>
                    </div>
                    <div class="mt-4 w-1/2">
                        <ftb-button color="warning" class="text-center px-2 py-1 my-2" @click="logout">Logout</ftb-button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import {readdirSync, readFileSync, existsSync} from 'fs';
import {createPaste} from 'hastebin';
import {Component, Vue, Watch} from 'vue-property-decorator';
import PackCardWrapper from '@/components/packs/PackCardWrapper.vue';
import FTBInput from '@/components/FTBInput.vue';
import FTBToggle from '@/components/FTBToggle.vue';
import FTBButton from '@/components/FTBButton.vue';
import FTBSlider from '@/components/FTBSlider.vue';
import {State, Action} from 'vuex-class';
import config from '@/config';
import {ipcRenderer, clipboard} from 'electron';
import path from 'path';
import { logVerbose } from '../utils';
import { AuthState } from '../modules/auth/types';
import { SettingsState } from '../modules/settings/types';

@Component({
    components: {
        'ftb-input': FTBInput,
        'ftb-toggle': FTBToggle,
        'ftb-slider': FTBSlider,
        'ftb-button': FTBButton,
        PackCardWrapper,
    },
})
export default class ProfilePage extends Vue {
    @State('auth')
    private auth!: AuthState;
    @State('settings')
    private settings!: SettingsState;
    @Action('logout', {namespace: 'auth'})
    private logoutAction!: () => void;
    @Action('saveSettings', {namespace: 'settings'}) public saveSettings: any;

    public async created() {
    
    }

    get avatarName(){
        let provider = this.auth.token?.accounts.find((s) => s.identityProvider === 'mcauth');
        return provider !== undefined && provider !== null ? provider.userId : "MHF_Steve";
    }

    public toggleCloudSaves(value: boolean) {
        this.settings.settings.cloudSaves = value;
        this.saveSettings(this.settings.settings);
    }
    public toggleAutoOpenChat(value: boolean) {
        this.settings.settings.autoOpenChat = value;
        this.saveSettings(this.settings.settings);
    }

    public openSubscriptions(){
        ipcRenderer.send('openLink', "https://minetogether.io/profile/subscriptions");
    }

    public openProfile(){
        ipcRenderer.send('openLink', "https://minetogether.io/profile/edit");
    }

    public logout(){
        this.logoutAction();
        // get instances and store
        ipcRenderer.send('logout');
        this.$router.push('/');
    }
}
</script>
