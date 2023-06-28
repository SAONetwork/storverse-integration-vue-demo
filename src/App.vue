<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <div>
      <input v-model="nodeDid" placeholder="your node did" />
      <button @click="userlogin">Connect</button>
    </div>
    <div>
      <button @click="loadProfile">Load My Own Profile</button>
      <button @click="updateDataModelPermission">Grant Permission</button>
    </div>
    <div>
      <input v-model="did" placeholder="did" />
      <button @click="loadProfileByDid">Load Other User's Profile</button>
    </div>
    <p>Status: {{ connectStatus }}</p>
    <p>My DID: {{ myDid }}</p>
    <div v-if="showMyProfile" class="profile">
      <div v-if="myProfile">
        <p>My Profile</p>
        <p>Username: {{ myProfile.username }}</p>
        <p>Twitter: {{ myProfile.twitter }}</p>
        <p>YouTube: {{ myProfile.youtube }}</p>
        <img :src="myAvatar" alt="My avatar" />
      </div>
      <button @click="showInputs = !showInputs">
        {{ showInputs ? 'Hide' : 'Edit Profile' }}
      </button>

      <div v-if="showInputs">
        <input v-model="username" placeholder="Username" />
        <input v-model="bio" placeholder="Bio" />
        <input v-model="twitter" placeholder="Twitter" />
        <input v-model="youtube" placeholder="YouTube" />
        <button @click="updateProfile">Update Profile</button>
      </div>
    </div>
    <div v-if="showOtherUserProfile" class="profile">
      <div v-if="otherUserProfile"> 
        <p>Load other user's Profile</p>
        <p>Username: {{ otherUserProfile.username }}</p>
        <p>Twitter: {{ otherUserProfile.twitter }}</p>
        <p>YouTube: {{ otherUserProfile.youtube }}</p>
        <img :src="otherUserAvatar" alt="Other user's avatar" />
      </div>
    </div>
  </div>
</template>

<script>
import { ref, reactive } from 'vue';
import { StorverseService, handleConnect, getConnectedWallet, SaoConfig} from "storverse-sdk";

export default {
  setup() {
    const nodeDid = ref('');
    const did = ref('');
    const connectStatus = ref('');
    const myDid = ref('');
    const myProfile = reactive({});
    const myAvatar = ref('');
    const otherUserProfile = reactive({});
    const otherUserAvatar = ref('');
    const showMyProfile = ref(false);
    const showOtherUserProfile = ref(false);

    const username = ref('');
    const bio = ref('');
    const twitter = ref('');
    const youtube = ref('');
    const showInputs = ref(false);

    let storverseService;

    async function userlogin() {
      const saoConfig = new SaoConfig(
        "sao-testnet1", // Chain ID of the network
        "Sao-Network", // Name of the blockchain network
        "https://api-testnet-node0.sao.network", // URL to the blockchain network's REST API endpoint
        "https://rpc-testnet-node0.sao.network", // URL to the blockchain network's RPC endpoint
        "http://43.156.118.116:5151/rpc/v0", // URL to the gateway's RPC endpoint
        "storverse-sao", // platform ID of storverse
        [nodeDid.value, 'did:key:zQ3shiAGhyFEGS3WhS64PYU9GEBk1rtrzaApJbHFEmWQbp5Xg'], // did of your node
        "https://graphql.storverse.app/graphql/query"
      );

      await handleConnect(saoConfig);

      const connectedWallet = getConnectedWallet();
      
      myDid.value = connectedWallet.sid;
      storverseService = new StorverseService(connectedWallet.modelManager, saoConfig);
      connectStatus.value = 'Connected';
      showMyProfile.value = false;
      showOtherUserProfile.value = false;
    }

    async function updateDataModelPermission() {
      await storverseService.ShareUserProfile();
    }

    async function loadDid() {
      const connectedWallet = getConnectedWallet();
      myDid.value = connectedWallet.sid;
      showMyProfile.value = false;
      showOtherUserProfile.value = false;
    }

    async function loadProfile() {
      var profile = await storverseService.GetUserProfile(myDid.value);
      console.log(profile);
      if (profile.avatarContent) {
        myAvatar.value = `data:image/png;base64,${profile.avatarContent}`;
      } else {
        console.log("Avatar is not defined in the profile.");
      }
      Object.assign(myProfile, profile);
      showMyProfile.value = true;
    }

    async function loadProfileByDid() {
      var profile = await storverseService.GetUserProfileDelegate(did.value);
      Object.assign(otherUserProfile, profile);
      if (profile.avatarContent) {
        otherUserAvatar.value = `data:image/png;base64,${profile.avatarContent}`;
      } else {
        console.log("Avatar is not defined in the profile.");
      }
      showOtherUserProfile.value = true;
    }

    async function updateProfile() {
      const profile = {
        id: myProfile.id,
        createdAt: myProfile.createdAt,
        updatedAt: Date.now(),
        did: myProfile.did,
        ethAddr: myProfile.ethAddr,
        avatar: myProfile.avatar,
        avatarContent: myProfile.avatarContent,
        username: username.value,
        twitter: twitter.value,
        youtube: youtube.value,
        bio: bio.value,
        banner: myProfile.banner
      };

      const dataId = await storverseService.SaveUserProfile(profile);
      console.log('Profile updated. Data ID:', dataId);
      await loadProfile(); // reload profile after update
    }

    return {
      nodeDid,
      did,
      connectStatus,
      myDid,
      myProfile,
      myAvatar,
      otherUserProfile,
      otherUserAvatar,
      showMyProfile,
      showOtherUserProfile,
      userlogin,
      updateDataModelPermission,
      loadDid,
      loadProfile,
      loadProfileByDid,
      username,
      bio,
      twitter,
      youtube,
      updateProfile,
      showInputs
    }
  }
}
</script>


<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  padding: 20px;
  border: 1px solid #2c3e50;
  border-radius: 10px;
  max-width: 800px;
  margin: 60px auto;
  background-color: #f5f5f5;
}

img {
  width: 100px;
  height: 100px;
  border-radius: 50%;  /* Added this to make the avatars circular */
  object-fit: cover;   /* Added this to make sure the images fit the space */
  margin-bottom: 20px; /* Added this to create some space between the image and text */
}

.container {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  margin: 20px;
}

button {
  background-color: #2c3e50;
  color: white;
  border: none;
  border-radius: 10px;
  margin: 10px;
  padding: 10px;
  transition: background-color 0.2s; /* Added this for a transition effect on hover */
}

input {
  border: 1px solid #2c3e50;
  border-radius: 10px;
  margin: 10px;
  padding: 10px;
}

button:hover {
  background-color: #1a2433; /* Darkened the background color on hover */
  cursor: pointer; /* Changed the cursor to a pointer on hover */
}

input:focus {
  outline-color: #2c3e50;
}

.profile {
  background-color: #ffffff;
  padding: 20px;
  margin: 20px;
  border: 1px solid #2c3e50;
  border-radius: 10px;
}
</style>

