<template>
  <div class="md-layout md-gutter" v-if="service == ''" style="padding: 30px;">
    <!--
      ----------------------------------------------------------------------
        SNACKBARS  - https://vuematerial.io/components/snackbar
      ----------------------------------------------------------------------
    -->
    <md-snackbar :md-duration="4000" :md-active.sync="showSnackBar" md-persistent>
      <span>{{ snack }}</span>
      <md-button class="md-primary" @click="showSnackbar = false;">Dismiss</md-button>
    </md-snackbar>

    <!--
      ----------------------------------------------------------------------
        DIALOG BOXES - EDIT CARD
      ----------------------------------------------------------------------
    -->
    <md-button
      v-if="!activeDialogchannel"
      @click="createCard();"
      class="md-fab md-primary"
      style="position: absolute; bottom: 20px; right: 20px; z-index: 99"
    >
      <md-icon>add</md-icon>
    </md-button>

    <md-dialog
      :md-close-on-esc="false"
      :md-click-outside-to-close="false"
      :md-active.sync="activeDialogchannel"
    >
      <md-dialog-title>
        <md-icon style="color: black;">group_work</md-icon>
        {{ mode }} Interest
        Group
      </md-dialog-title>

      <!-- https://vuematerial.io/components/form name="dialog.cardForm" -->
      <div style="padding: 0 25px ;">
        <md-field id="display_name">
          <label>Title</label>
          <md-input v-on:keyup="slug" v-model="display_name" required></md-input>
          <span class="md-helper-text">Enter the name of this Interest Group</span>
          <span class="md-error">This field cannot be blank</span>
        </md-field>

        <md-field id="name">
          <label>Slug</label>
          <md-input v-model="name" required></md-input>
          <span class="md-helper-text">Unique name as it appears in the URL</span>
          <span class="md-error">This field cannot be blank</span>
        </md-field>

        <md-field>
          <label>Icon</label>
          <md-input v-model="icon" required></md-input>
          <span class="md-helper-text">
            Pick an icon
            <a
              href="https://material.io/tools/icons/?style=baseline"
              target="icons"
            >from this list</a>
          </span>
          <span class="md-error"></span>
          <md-icon>{{ icon }}</md-icon>
        </md-field>

        <md-field>
          <label>Description</label>
          <md-textarea style="font-size: 0.9em;" v-model="header" required></md-textarea>
          <span class="md-helper-text">Enter a short description for this Interest Group</span>
          <span class="md-error">This field cannot be blank</span>
        </md-field>

        <md-field id="formtags">
          <vue-tags-input
            :validation="validation"
            style="ma rgin-top: 50px; width: 100%; border: 0px"
            required="true"
            v-model="tag"
            @tags-changed="newTags => (formtags = newTags)"
            :tags="formtags"
            :allow-edit-tags="true"
            :autocomplete-items="autocompleteItems"
          ></vue-tags-input>
          <span class="md-helper-text">Enter one or more tags</span>
          <span class="md-error">This field cannot be blank</span>
        </md-field>

        <md-dialog-actions style="padding: 25px 0;">
          <md-button class="md-success md-raised" @click="activeDialogchannel = false;">Cancel</md-button>
          <md-button
            v-if="mode == 'Edit'"
            class="md-success md-raised"
            @click="onConfirm(formindex);"
            style="background: #0DC9C9; color: white;"
          >Update</md-button>
          <md-button
            v-if="mode == 'Create'"
            class="md-success md-raised"
            @click="onConfirm();"
            style="background: #0DC9C9; color: white;"
          >Create</md-button>
        </md-dialog-actions>
      </div>
    </md-dialog>

    <!--
      ----------------------------------------------------------------------
        CARDS
      ----------------------------------------------------------------------
    -->
    <md-card
      md-with-hover
      v-for="(channel, index) in members(channels)"
      :key="index"
      class="md-layout-item"
    >
      <div class="md-card-banner">
        <md-menu>
          <md-button class="md-icon-button" md-menu-trigger>
            <md-icon style="font-size: 1.8em !important; color: white;">menu</md-icon>
          </md-button>

          <md-menu-content class="md-card-menu">
            <md-menu-item @click="editCard(channel, index);">
              <md-icon>edit</md-icon>
              <span>Edit</span>
            </md-menu-item>

            <md-menu-item @click="deleteCard(channel);">
              <md-icon>archive</md-icon>
              <span>Archive</span>
            </md-menu-item>
          </md-menu-content>
        </md-menu>

        <div class="md-subhead">{{ channel.display_name[0] }} Channel</div>
        <div class="pin" @click="switchFavorite(channel.channel_id);">
          <md-icon v-if="isFavorite(channel.channel_id)">favorite</md-icon>
          <md-icon v-else>favorite_border</md-icon>
        </div>
      </div>

      <div class="md-card-header">
        <md-card-header-text>
          <div class="md-title">
            <md-icon
              style="line-height: 0.9; font-size: 1em !important; color: #404040;"
            >{{ channel.purpose.icon }}</md-icon>
            {{ channel.display_name.replace(/[!#*@%/."'\\&]/, "") }}
          </div>
        </md-card-header-text>

        <md-chip class="md-status" style="background-color: green" v-if="isMember(channel)">Joined</md-chip>
        <md-chip
          class="md-status"
          style="background-color: orange"
          v-if="isSuggested(channel) && !isMember(channel)"
        >Suggested</md-chip>
        <md-chip
          class="md-status"
          style="background-color: #ccc"
          v-if="!isMember(channel) && !isSuggested(channel)"
        >Not Joined</md-chip>
      </div>

      <div class="md-card-mid">
        <p class="info md-scrollbar">
          <vue-markdown>{{channel.header}}</vue-markdown>
        </p>
      </div>

      <md-card-actions>
        <md-button v-if="isMember(channel)" @click="cardAction('leave', channel);">Leave</md-button>
        <md-button
          v-if="isMember(channel)"
          style="background: #0DC9C9; color: white;"
          @click="cardAction('open', channel);"
        >Open</md-button>
        <md-button v-if="!isMember(channel)" @click="cardAction('ask', channel);">Ask</md-button>
        <md-button
          v-if="!isMember(channel)"
          style="background: #0DC9C9; color: white;"
          @click="cardAction('join', channel);"
        >Join</md-button>
      </md-card-actions>

      <div class="md-card-footer">
        <div class="md-card-avatars md-scrollbar">
          <md-avatar v-for="(member, index) in channel.members" :key="index">
            <img v-bind:src="avatarLink(member)" alt="Avatar">
            <md-tooltip md-direction="top">{{ member }}</md-tooltip>
          </md-avatar>
        </div>
      </div>
    </md-card>
  </div>
</template>

<script>
import VueTagsInput from "@johmun/vue-tags-input";
import VueMarkdown from "vue-markdown";
import Slack from "node-slack";
import Slugify from "slugify";
import _ from "lodash/fp/object"; //lodash/fp/object for objects only
import { BASEURL, CHATURL } from "../main.js";
import db from "../firebase/init.js";

export default {
  name: "Tags",
  components: { VueTagsInput, VueMarkdown },
  props: ["domain"],
  data() {
    return {
      service: "",
      tag: "",
      mode: "", // Edit or Create mode for dialog
      script: "", // Edit or Create script for execution
      showServices: false,
      showCardNavigation: false,
      activeDialogchannel: false,
      showSnackBar: false,
      snack: "",
      result: "",
      status: "",
      channels: [],
      //domain: "",
      domains: [],
      channels2: [],
      groups: [],
      tags: [],
      favorites: [],
      autocompleteItems: [
        { text: "Accessibility", frequency: 5 },
        { text: "Accounting", frequency: 2 },
        { text: "Administration", frequency: 4 },
        { text: "Artificial Intelligence", frequency: 5 },
        { text: "Authentication", frequency: 7 },
        { text: "Automation", frequency: 4 },
        { text: "Biometrics", frequency: 10 },
        { text: "Bitcoin", frequency: 10 }
      ],
      channel_id: "",
      display_name: "",
      name: "", //Slugify(this.display_name),
      header: "",
      icon: "",
      formtags: [],
      formindex: "",
      username: "",
      validation: [
        {
          type: "min-length",
          rule: /^.{3,}$/,
          disableAdd: true
        },
        {
          type: "no-numbers",
          rule: /^([^0-9]*)$/
        },
        {
          type: "avoid-item",
          rule: /^(?!Cannot).*$/,
          disableAdd: true
        },
        {
          type: "no-braces",
          rule: text => text.indexOf("{") !== -1 || text.indexOf("}") !== -1
        }
      ]
    };
  },
  ///////////////////////////////////////////////////////////////////////////////
  //  CREATED - https://vuejs.org/v2/guide/instance.html
  ///////////////////////////////////////////////////////////////////////////////
  created: function() {
    this.username = this.$cookies.get("username").replace(".", "%2E");
    // instant relaod does not refresh domain prop in time
    if (this.domain === "") {
      this.domain = this.$route.query.domain;
    }

    // LOAD USER GROUPS AND TAGS /////////////////////////////////////////////
    if (this.username) {
      let groupsRef = db.database().ref("portal_profiles/" + this.username);
      groupsRef.on("child_added", group => {
        var data = group.val();
        if (group.key === "channels") {
          this.groups = data;
        } else if (group.key === "tags") {
          for (const key of data) {
            this.tags.push(key.text);
          }
          // convert tags to flat array
          // this.tags =
          //   //fails with 1 data object
          //   data.reduce((accumulator, currentValue) => {
          //     return [...accumulator, currentValue.text];
          //   });
        }
      });
    }

    // LOAD USER PREFERENCES ///////////////////////////////////////////////////
    let prefsRef = db.database().ref("portal_profiles/" + this.username);

    prefsRef.on("child_added", pref => {
      if (pref.key === "prefs") {
        this.showServices = pref.val().showServices;
      } else if (pref.key === "favorites") {
        this.favorites = pref.val();
      }
    });

    prefsRef.on("child_changed", pref => {
      if (pref.key === "prefs") {
        this.showServices = pref.val().showServices;
      } else if (pref.key === "favorites") {
        this.favorites = pref.val();
      }
    });

    // LOAD DOMAINS AND CHANNELS /////////////////////////////////////////////
    let channelsRef = db.database().ref("portal_channels");
    let domainsRef = db
      .database()
      .ref("portal_profiles/" + this.username + "/domains");

    //console.log(this.domains, domainsRef)
    domainsRef
      .once("value", profile => {
        if (profile.exists()) {
          this.domains = profile.val();
        }
      })
      .then(profile => {
        // // porta_extensions contains any channel info not stored in Mattermost
        channelsRef.on("child_added", channel => {
          let data = channel.val();
          //console.log(this.domain, this.domains);
          //console.log(this.domains, data.team)
          if (
            this.domain === data.team ||
            (this.domain === "all" && this.domains.includes(data.team))
          ) {
            this.channels.push(data);

            //this.channels.sort(SortByName);  WARNING: SORT CORRUPTS DATA
            // var data = _.sortByOrder(array_of_objects, ['type','name'], [true, 'desc']);
          }
        });
      });

    channelsRef.on("child_changed", channel => {
      let data = channel.val();
      this.channels.forEach(function(element, index, arr) {
        if (element.channel_id === data.channel_id) {
          // lodash function to merge objects recursively
          arr[index] = _.merge(arr[index], data);
        }
      });
    });

    channelsRef.on("child_removed", channel => {
      let data = channel.val();
      this.channels.forEach(function(element, index, arr) {
        if (element.channel_id === data.channel_id) {
          // lodash function to merge objects recursively
          arr.splice(index, 1);
        }
      });
    });

    //this.$forceUpdate();
  },
  ///////////////////////////////////////////////////////////////////////////////
  //  COMPUTED - https://vuejs.org/v2/guide/instance.html
  ///////////////////////////////////////////////////////////////////////////////
  computed: {
    // invalidate: function() {
    //   return this.invalid === true ? "md-invalid" : "";
    // }
  },
  ///////////////////////////////////////////////////////////////////////////////
  //  METHODS - https://vuejs.org/v2/guide/instance.html
  ///////////////////////////////////////////////////////////////////////////////
  methods: {
    // is member
    members: function(channels) {
      return channels.filter(channel => {
        return (
          (this.showServices &&
            JSON.stringify(this.groups).includes(
              channel.team + "/" + channel.name
            )) ||
          !this.showServices
        );
      });
    },

    switchFavorite: function(id) {
      if (this.favorites.includes(id)) {
        for (var i = 0; i < this.favorites.length; i++) {
          if (this.favorites[i] === id) {
            this.favorites.splice(i, 1);
          }
        }
      } else {
        this.favorites.push(id);
      }
      let profileRef = db.database().ref("portal_profiles/" + this.username);
      profileRef.update({ favorites: this.favorites });
    },

    isFavorite: function(id) {
      return this.favorites.includes(id);
    },

    isMember: function(channel) {
      return JSON.stringify(this.groups).includes(
        channel.team + "/" + channel.name
      );
    },
    // is suggested
    isSuggested: function(channel) {
      if (channel.purpose.tags && this.tags) {
        //alert(this.tags);
        return this.tags.filter(
          value => -1 !== channel.purpose.tags.indexOf(value)
        ).length;
      } else {
        return false;
      }
    },

    // create slug for URL
    slug: function() {
      this.name = Slugify(this.display_name, { lower: true });
    },
    // compute v-bind:src for img
    avatarLink: function(username) {
      return BASEURL + "images/avatars/avatar_" + username + ".png";
    },

    // create new card
    createCard: function(channel) {
      // all interest groups created in the diglife domain
      this.team_id = this.channels[0].team_id;
      this.channel_id = "";
      this.display_name = "";
      this.name = "";
      this.icon = "";
      this.header = "";
      this.formtags = [];
      this.mode = "Create";
      this.activeDialogchannel = true;
    },

    // edit existing card (needs index of channel)
    editCard: function(channel, index) {
      //alert(channel.purpose.tags);
      this.team_id = channel.team_id;
      this.channel_id = channel.channel_id;
      this.display_name = channel.display_name;
      this.name = channel.name;
      this.icon = channel.purpose.icon;
      this.header = channel.header;
      //this.formtags = channel.purpose.tags;
      // the tag control needs text key-value pairs
      if (channel.purpose.tags && channel.purpose.tags[0].text === undefined) {
        this.formtags = channel.purpose.tags.map(function(element) {
          return { text: element };
        });
      } else {
        this.formtags = channel.purpose.tags;
      }
      this.formindex = index;
      this.mode = "Edit";
      this.activeDialogchannel = true;
    },

    // edit existing card (needs index of channel)
    deleteCard: function(channel, index) {
      this.axios
        .get(
          BASEURL +
            "webhooks/" +
            "portal_delete_channel.php" +
            "?file=base-diglife-coop.php" +
            "&channel_id=" +
            channel.channel_id
        )
        .then(delete this.channels[index])
        .then(
          db
            .database()
            .ref("portal_channels/" + channel.channel_id)
            .remove()
        )
        .then(
          db
            .database()
            .ref("portal_extensions/" + channel.channel_id)
            .remove()
        )
        .then(response => {
          this.showSnackBar = true;
          this.snack = "This card has been successfully removed.";
        })
        .catch(error => {
          this.showSnackBar = true;
          this.snack = "Network Error, please try again later.";
          console.log(error);
        });
    },

    // submit card edits
    onConfirm: function(formindex) {
      // error validation
      if (this.display_name === "") {
        document.getElementById("display_name").classList.add("md-invalid");
      } else if (this.name === "") {
        document.getElementById("name").classList.add("md-invalid");
      } else if (this.formtags.length === 0) {
        // not working -- need to invoke requored field here
        document.getElementById("formtags").classList.add("md-invalid");
        // no errors
      } else {
        // we have two scripts for updating and creating channels
        if (this.mode === "Edit") {
          this.script = "portal_update_channel.php";
        } else if (this.mode === "Create") {
          this.script = "portal_create_channel.php";
        }

        // prepare some values before writing them back to MM
        // channel channels starting with "#"
        if (this.display_name[0] !== "#") {
          this.display_name = "#" + this.display_name;
        }
        this.formtags.unshift({ text: "test" }); // It's missing the first element, so faking it
        this.formtags = this.formtags.reduce(function(
          accumulator,
          currentValue
        ) {
          return [...accumulator, currentValue.text];
        });

        //console.log(this.formtags);
        this.axios
          .get(
            BASEURL +
              "webhooks/" +
              this.script +
              "?file=base-diglife-coop.php" +
              "&channel_id=" +
              this.channel_id +
              "&team_id=" +
              this.team_id +
              "&display_name=" +
              this.display_name.replace("#", "%23") +
              "&name=" +
              this.name +
              "&header=" +
              encodeURI(this.header) +
              "&icon=" +
              this.icon +
              "&tags=" +
              JSON.stringify(this.formtags)
          )
          // does not update all fields
          .then(response => {
            if (this.mode === "Edit") {
              this.channel = _.merge(this.channels[formindex], response.data);
            } else {
              this.channel = response.data;
              // this would be pushed twice, child_added
              //this.channels.push(response.data);
            }
          })

          .then(response =>
            db
              .database()
              .ref("portal_channels/" + this.channel.id)
              .update({
                name: this.channel.name,
                display_name: this.channel.display_name,
                header: this.channel.header,
                purpose: JSON.parse(this.channel.purpose)
              })
          )
          .then(response => {
            this.showSnackBar = true;
            this.snack = "This card has been successfully updated.";
          })
          .catch(error => {
            if (this.channel.status_code) {
              this.snack = "Mattermost Error: " + this.channel.message;
            } else {
              this.snack = "Network Error, please try again later.";
            }
            this.showSnackBar = true;
          });

        this.activeDialogchannel = false;
      }
    },

    // execute card action
    cardAction: function(action, channel) {
      switch (action) {
        case "join":
          // join channel
          this.axios
            .get(
              BASEURL +
                "webhooks/portal_join_channel.php?file=base-diglife-coop.php&username=" +
                this.$cookies.get("username") +
                "&channel_id=" +
                channel.channel_id
            )
            .then(response => (this.result = response.data))
            .then(response =>
              this.groups.push(channel.team + "/" + channel.name)
            )
            .then(response =>
              channel.members.push(this.$cookies.get("username"))
            );
          break;
        case "open":
          // following not working since service is not visible in Navbar
          // window.onload = function() {
          //   window.open(channel.purpose.link, "theApp");
          this.service = channel.display_name.replace("#", "");
          var element = document.getElementById("theApp");
          element.src = "about:blank";
          element.style.display = "block";
          window.open(
            CHATURL + this.domain + "/channels/" + channel.name,
            "theApp"
          );
          // };
          break;
        case "ask":
          let question = prompt("Please ask your question", "Ask the Expert");

          var slack = new Slack(CHATURL + "hooks/" + channel.purpose.hook);
          var err = slack.send({
            text:
              "##### :question: Question from @" +
              this.username +
              "\n" +
              question +
              "\n_Tip: you can respond via a direct message or invite the user into this channel._",
            channel: channel.name,
            username: "ledgerbot",
            icon_url: "https://diglife.com/brand/logo_secondary_dark.svg",
            unfurl_links: true,
            link_names: 1
          });
          this.snack = "Thank You. Your question has been submitted.";
          this.showSnackBar = true;

          break;
        case "leave":
          // leave channel
          this.axios
            .get(
              BASEURL +
                "webhooks/portal_leave_channel.php?file=base-diglife-coop.php&username=" +
                this.$cookies.get("username") +
                "&channel_id=" +
                channel.channel_id
            )
            .then(response => (this.result = response.data))
            .then(response =>
              this.groups.splice(
                this.groups.indexOf(channel.team + "/" + channel.name),
                1
              )
            )
            .then(response =>
              channel.members.splice(
                channel.members.indexOf(this.$cookies.get("username")),
                1
              )
            );
          break;
        default:
      }
    }
  }
};
</script>
<style>
.pin {
  position: absolute !important;
  right: 10px !important;
  top: 10px !important;
  cursor: pointer;
}

.pin .md-icon {
  font-size: 2em !important;
  color: white !important;
}
</style>
