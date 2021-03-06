<template>
  <div class="container-xl px-0">
    <div id="navigation-top" class="d-flex justify-content-end">
      <b-btn
        @click="loginAction"
        variant="white"
        class="block-rounded"
        size="sm"
        v-b-tooltip.hover
        :title="loginTooltip"
      >
        <span class="d-none d-sm-inline mr-2">{{ loginButtonLabel }}</span>
        <font-awesome-icon :icon="icon" :pulse="spin" fixed-width />
      </b-btn>
      <settings-menu />
      <bancor-menu />
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Watch } from "vue-property-decorator";
import { vxm } from "@/store/";
import SettingsMenu from "@/components/layout/SettingsMenu.vue";
import BancorMenu from "@/components/layout/BancorMenu.vue";
import { shortenEthAddress } from "@/api/helpers";
import BaseComponent from "@/components/BaseComponent.vue";

@Component({
  components: { BancorMenu, SettingsMenu }
})
export default class Navigation extends BaseComponent {
  get selectedWallet() {
    return vxm.wallet.currentWallet;
  }

  created() {
    vxm.ethWallet.checkAlreadySignedIn();
  }

  @Watch("currentUser")
  onAuthentication(account: string) {
    if (account) {
      vxm.bancor.refreshBalances();
    }
  }

  get loginTooltip() {
    return this.currentNetwork == "eth" && this.currentUser
      ? "Logout via wallet"
      : "";
  }

  get loginStatus() {
    return vxm.eosWallet.loginStatus;
  }

  get shortenedEthAddress() {
    const currentUser = vxm.ethWallet.currentUser;
    return currentUser.length > 13
      ? shortenEthAddress(currentUser)
      : currentUser;
  }

  get loginButtonLabel() {
    if (this.selectedWallet == "eos") {
      return this.loginStatus[0];
    } else {
      const currentUser = vxm.ethWallet.currentUser;
      if (currentUser) {
        return this.shortenedEthAddress;
      } else return "Connect Wallet";
    }
  }

  get icon() {
    if (this.selectedWallet == "eos") {
      return this.loginStatus[1];
    } else {
      return vxm.ethWallet.currentUser ? "power-off" : "arrow-circle-right";
    }
  }

  get spin() {
    return this.loginStatus[2];
  }

  async loginActionEos() {
    const status = this.loginButtonLabel;
    if (status === "Connect Wallet") {
      this.$bvModal.show("modal-login");
    } else if (
      status !== "Authenticating" &&
      status !== "Connecting" &&
      status !== "Fetching"
    ) {
      vxm.eosWallet.logout();
    }
  }

  async loginActionEth() {
    if (vxm.ethWallet.currentUser) {
      // Cannot logout of MetaMask
    } else {
      await vxm.ethWallet.connect();
    }
  }

  async loginAction() {
    const wallet = this.selectedWallet;
    if (wallet == "eos") await this.loginActionEos();
    else await this.loginActionEth();
  }
}
</script>

<style lang="scss">
#navigation-top {
  margin: 1rem 0;

  & > * {
    margin-left: 0.75rem;
  }
}
</style>
