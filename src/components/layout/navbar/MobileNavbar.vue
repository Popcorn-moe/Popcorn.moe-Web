<template>
	<div>
		<v-toolbar app dense style="z-index: 4;">
			<v-slide-y-transition mode="out-in">
				<div v-if="search" key="search" class="toolbar__content" style="width: 100%;">
					<v-btn @click.stop="search = false, query = null" flat icon>
						<v-icon>arrow_back</v-icon>
					</v-btn>
					<v-text-field
					 :label="$t('mobile_navbar.search')"
					 hide-details
					 single-line
					 v-model="query"
					></v-text-field>
				</div>
				<div v-else key="toolbar" class="toolbar__content" style="width: 100%;"> 
					<object alt="logo" height="40px" data="/static/logo-animated.svg" type="image/svg+xml"></object>
					<v-spacer></v-spacer>
					<v-btn icon @click.stop="search = true">
						<v-icon>search</v-icon>
					</v-btn>
					<v-dialog 
					 v-if="isAuth && me"
					 fullscreen
        			 transition="dialog-bottom-transition"
        			 :overlay="false"
        			 scrollable
					 v-model="dialog"
					>
						<v-avatar size="36px" slot="activator">
              				<img :src="me.avatar">
            			</v-avatar>
						<v-card tile>
							<v-toolbar dense card color="primary">
								<v-btn icon @click.native="dialog = false" dark>
									<v-icon>close</v-icon>
								</v-btn>
								<v-toolbar-title v-t="'mobile_navbar.account'"></v-toolbar-title>
							</v-toolbar>
							<v-list>
								<auth-menu-links @close="dialog = false"></auth-menu-links>
							</v-list>
						</v-card>
					</v-dialog>
					<v-btn v-else icon :to="{ name: 'Login' }">
						<v-icon>account_circle</v-icon>
					</v-btn>
				</div>
			</v-slide-y-transition>
		</v-toolbar>
		<search-results class="floating-search-results" @close="search = false, query = null" :query="query"></search-results>
		<v-bottom-nav app :value="true">
			<v-btn
			 flat
			 v-for="(route, i) in routes.filter(r => !r.hide)"
			 :key="i"
			 :to="route.path"
			 color="primary"
			>
				<span v-t="route.t"></span>
				<v-icon v-html="route.icon"></v-icon>
			</v-btn>
		</v-bottom-nav>
	</div>
</template>

<script>
import {
	VBottomNav,
	VBtn,
	VIcon,
	VTextField,
	VAvatar,
	VDialog,
	VCard,
	VList
} from "vuetify/es5/components";
import { VSpacer } from "vuetify/es5/components/VGrid";
import { VToolbar, VToolbarTitle } from "vuetify/es5/components/VToolbar";
import { VSlideYTransition } from "vuetify/es5/components/transitions";
import SearchResults from "../SearchResults";
import AuthMenuLinks from "./AuthMenuLinks";
import { routes } from "../../../router";
import { mapGetters, mapActions } from "vuex";
import gql from "graphql-tag";

export default {
	data() {
		return {
			routes,
			query: null,
			search: false,
			dialog: false,
			me: null
		};
	},
	components: {
		VBottomNav,
		VBtn,
		VIcon,
		VSpacer,
		VToolbar,
		VToolbarTitle,
		VSlideYTransition,
		VTextField,
		VAvatar,
		VDialog,
		VCard,
		SearchResults,
		VList,
		AuthMenuLinks
	},
	computed: mapGetters({
		darkTheme: "darkTheme",
		isAuth: "isAuth"
	}),
	methods: mapActions({
		setDarkTheme: "setDarkTheme"
	}),
	watch: {
		isAuth() {
			this.$apollo.queries.me.refetch();
		}
	},
	apollo: {
		me: {
			query: gql`
				{
					me {
						id
						avatar
						login
					}
				}
			`,
			update: ({ me }) => me
		}
	},
	i18n: {
		messages: {
			fr: {
				mobile_navbar: {
					search: "Rechercher",
					account: "Mon compte"
				}
			},
			en: {
				mobile_navbar: {
					search: "Search",
					account: "Account"
				}
			}
		}
	}
};
</script>

<style lang="stylus">
  @import '../../../stylus/main'

  .floating-search-results {
	  position:	fixed;
	  margin-top: 48px;
	  z-index: 4;
	  width: 100%;
  }
</style>