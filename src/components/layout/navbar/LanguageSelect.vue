<template>
	<v-select
		:items="Object.keys(i18n.messages)"
		@input="setLocale"
		:value="i18n.locale.split('-', 1)[0]"
		hide-details
		:light="!transparent"
		:solo="!transparent"
		:color="!transparent ? 'black' : 'inherit'"
	>
		<template slot="selection" slot-scope="data">
			<v-avatar size="25px">
				<img :src="`https://cdn.rawgit.com/lipis/flag-icon-css/fe79c175/flags/1x1/${MAP[data.item] || data.item }.svg`"/>
			</v-avatar>
			<span style="margin-left: 10px" v-text="$t('language_select.name')"></span>
		</template>
		<template slot="item" slot-scope="data">
			<v-list-tile-avatar>
				<img class="language-flag" :src="`https://cdn.rawgit.com/lipis/flag-icon-css/fe79c175/flags/1x1/${MAP[data.item] || data.item }.svg`"/>
			</v-list-tile-avatar>
			<v-list-tile-content>
				<v-list-tile-title v-html="$t('language_select.name', data.item)"></v-list-tile-title>
			</v-list-tile-content>
		</template>
	</v-select>
</template>

<script>
import { VSelect, VAvatar } from "vuetify";
import {
	VListTileAvatar,
	VListTileContent,
	VListTileTitle
} from "vuetify/es5/components/VList";
import i18n from "../../../i18n";
const MAP = {
	en: "gb"
};

export default {
	props: {
		transparent: {
			type: Boolean,
			default: true
		}
	},
	data() {
		return {
			MAP,
			i18n
		};
	},
	methods: {
		setLocale(locale) {
			this.$emit("change");
			i18n.locale = locale;
			localStorage.setItem("locale", locale);
			const update = elem => {
				elem.$forceUpdate();
				elem.$children.forEach(update);
			};

			update(this.$root);
		}
	},
	components: {
		VSelect,
		VAvatar,
		VListTileAvatar,
		VListTileContent,
		VListTileTitle
	},
	i18n: {
		messages: {
			fr: {
				language_select: {
					name: "Français"
				}
			},
			en: {
				language_select: {
					name: "English"
				}
			}
		}
	}
};
</script>

<style>
.language-flag {
	width: 35px !important;
	height: 35px !important;
}
</style>
