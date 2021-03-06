<template>
  <loader v-if="loading"></loader>
  <div v-else>
    <div class="anime-page-banner" :style="{ 'background-image': `url(${anime.background})` }">
			<v-container >
				<v-layout row>
        	<v-flex offset-sm1 sm10>
						<v-speed-dial
							v-if="user.id != null"
							v-model="fab"
							direction="left"
							transition="slide-x-reverse-transition"
							open-on-hover
							top
							left
						>
							<v-tooltip slot="activator" top>
								<v-btn
									class="main"
									slot="activator"
									v-model="fab"
									:color="statusButtons[animeStatus] && statusButtons[animeStatus].color || 'primary'"
									fab
								>
									<v-icon>{{ statusButtons[animeStatus] && statusButtons[animeStatus].icon || 'library_books' }}</v-icon>
									<v-icon>close</v-icon>
								</v-btn>
								<span>Library</span>
							</v-tooltip>
							<div v-for="(btn, status) in statusButtons" :key="status">
								<v-tooltip top>
									<v-btn
										fab
										small
										slot="activator" 
										:color="animeStatus !== status && btn.color || 'grey'"
										@click="animeStatus !== status && setAnimeStatus(status)"
									>
										<v-icon>{{ btn.icon }}</v-icon>
									</v-btn>
									<span>{{ btn.tooltip }}</span>
								</v-tooltip>
							</div>
						</v-speed-dial>
        	</v-flex>
				</v-layout>
			</v-container>
		</div>
    <v-container class="anime-page-container">
      <v-layout row wrap>
        <v-flex offset-sm1 sm7 class="anime-infos">
          <p-img class="anime-page-cover" alt="cover" :src="anime.cover.normal"/>
          <h3 class="uppercase">{{ anime.names[0] }}</h3>
            <ul>
              <li>
                <div class="list-name" v-t="anime.names.length > 1 ? 'anime.names' : 'anime.name'"></div>
                {{ anime.names.join(', ') }}
              </li>
              <li>
                <div class="list-name" v-if="anime.authors.length" v-t="anime.authors.length > 1 ? 'anime.authors' : 'anime.author'"></div>
                <div  v-for="(author, i) in anime.authors" :key="i" style="display: inline">
									{{ i > 0 ? ',' : "" }}
									<router-link :to="{ name: 'Author', params: { id: author.id }}">
										{{ author.name }}
									</router-link>
								</div>
              </li>
              <li>
                <div class="list-name" v-if="anime.tags.length" v-t="'anime.tags'"></div>
                {{ anime.tags.map(({ name }) => name).join(', ') }}
              </li>
              <li>
                <div class="list-name" v-t="'anime.status'"></div>
                <div v-t="`anime_status.${anime.status}`"></div>
              </li>
            </ul>
          <p>{{ anime.desc }}</p>
          <div class="text-xs-center" v-if="trailers.length">
            <h4 class="uppercase" v-t="'anime.trailer'"></h4>
						<div class="videoWrapper">
							<iframe 
								width="100%" 
								:src="trailers[0].content" 
								frameborder="0" 
								allow="autoplay; encrypted-media" 
								allowfullscreen
							></iframe>
						</div>
          </div>
        </v-flex>
        <v-flex sm3 xs12>
          <div class="rate-container">
            <div class="text-xs-center">
              <h4 v-t="'anime.rating'"></h4>
              <rating v-model="anime.rating"></rating>
            </div>
          </div>
          <media-list :anime="anime"></media-list>
        </v-flex>
      </v-layout>
    </v-container>
  </div>
</template>

<script>
import Loader from "../components/layout/Loader";
import { VBtn, VIcon, VSpeedDial, VTooltip } from "vuetify";
import { VContainer, VFlex, VLayout } from "vuetify/es5/components/VGrid";
import Rating from "../components/Rating";
import MediaList from "../components/media/MediaList";
import PImg from "../components/ProgressiveImg";
import gql from "graphql-tag";
import { client } from "../graphql";

const statusButtons = {
	WANT_TO_WATCH: {
		icon: "watch_later",
		color: "blue",
		tooltip: "Want to watch"
	},
	COMPLETED: {
		icon: "check",
		color: "green",
		tooltip: "Completed"
	},
	ON_HOLD: {
		icon: "pause",
		color: "orange",
		tooltip: "On Hold"
	},
	DROPPED: {
		icon: "delete",
		color: "red",
		tooltip: "Dropped"
	},
	WATCHING: {
		icon: "play_arrow",
		color: "purple",
		tooltip: "Watching"
	}
};

export default {
	props: ["id"],
	data() {
		return {
			anime: null,
			user: {},
			fab: false,
			statusButtons,
			animeStatus: null
		};
	},
	watch: {
		anime() {
			this.$emit("updateHead");
		}
	},
	computed: {
		trailers() {
			return this.anime.medias.filter(({ type }) => type === "TRAILER");
		},
		loading() {
			return !this.anime;
		}
	},
	methods: {
		setAnimeStatus(status) {
			this.$apollo
				.mutate({
					mutation: gql`
						mutation($anime: ID!, $meta: MetaInput!) {
							updateMeta(anime: $anime, meta: $meta) {
								status
							}
						}
					`,
					variables: {
						anime: this.id,
						meta: { status }
					}
				})
				.then(
					({
						data: {
							updateMeta: { status }
						}
					}) => (this.animeStatus = status)
				);
		}
	},
	apollo: {
		anime: {
			query: gql`
				query($id: ID!) {
					anime(id: $id) {
						id
						names
						cover {
							normal
							big
						}
						background
						status
						desc
						tags {
							name
						}
						authors {
							id
							name
						}
						medias {
							id
							name
							type
							content
						}
						seasons {
							name
							episodes {
								id
								name
								content
							}
						}
					}
				}
			`,
			variables() {
				return {
					id: this.id
				};
			},
			update({ anime }) {
				if (!anime) {
					this.$router.replace({ name: "404" });
					return;
				}
				return anime;
			}
		},
		user: {
			query: gql`
				query($id: ID!) {
					me {
						id
						meta(anime: $id) {
							status
						}
					}
				}
			`,
			fetchPolicy: "network-only",
			variables() {
				return {
					id: this.id
				};
			},
			update({ me }) {
				this.animeStatus = me.meta && me.meta.status;
				return me;
			}
		}
	},
	components: {
		Loader,
		VContainer,
		VFlex,
		VLayout,
		VBtn,
		VIcon,
		Rating,
		MediaList,
		PImg,
		VSpeedDial,
		VTooltip
	},
	head: {
		title() {
			if (this.anime)
				return {
					inner: this.anime.names[0]
				};
		},
		meta() {
			if (this.anime)
				return [
					{
						property: "og:title",
						content: this.anime.names[0],
						id: "og:title"
					},
					{ property: "og:image", content: this.anime.big, id: "og:image" },
					{
						property: "og:description",
						content: this.anime.desc,
						id: "og:description"
					},
					{
						name: "description",
						content: this.anime.desc,
						id: "description"
					}
				];
		}
	},
	i18n: {
		messages: {
			fr: {
				anime: {
					subscribe: "S'abonner",
					name: "Nom :",
					names: "Noms :",
					author: "Auteur :",
					authors: "Auteurs :",
					tags: "Tags :",
					status: "Statut :",
					trailer: "Trailer :",
					rating: "Note :"
				}
			},
			en: {
				anime: {
					subscribe: "Subscribe",
					name: "Name:",
					names: "Names:",
					author: "Author:",
					authors: "Authors:",
					tags: "Tags:",
					status: "Status:",
					trailer: "Trailer:",
					rating: "Rating:"
				}
			}
		}
	}
};
</script>

<style lang="stylus">
  @import '../stylus/main.styl';

  .anime-page-banner {
    width: 100%;
    height: 405px;
		padding-top: 330px;
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center;
    background-color: #2f2f2f;

		.v-speed-dial {
			float: right
			.main{
				.v-icon {
					margin-top: -11px	
				}
			}
		}
  }

	

  .anime-page-container {
    padding-top:30px;

		.videoWrapper {
			position: relative;
			padding-bottom: 56.25%; /* 16:9 */
			padding-top: 25px;
			height: 0;

			iframe {
				position: absolute;
				top: 0;
				left: 0;
				width: 100%;
				height: 100%;
			}
		}
	

    .anime-infos {
      text-align: justify;
      padding-right: 10px !important

      h3 {
        margin-bottom: 10px !important
      }

      li {
        list-style: none;
        padding-right: 10px !important
      }

      .list-name {
        float: left;
        padding-right: 3px;
        font-weight: bold;
      }

      .list-name:before {
        float: right;

      }

      p {
        padding-top: 30px;
        padding-right: 10px;
      }
    }

    .anime-page-cover {
      margin-right: 18px;
      margin-bottom: 15px;
      width: 180px;
      height: 250px;
      margin-top: -105px;
      box-shadow: 0px 2px 12px 0px rgba(16,16,17,0.5);
      float: left;
    }

    .rate-container {
      background-color: #dcdcdc;
      margin-top: 30px;
      padding: 30px;
    }
  }

  .application.theme--dark .anime-page-container {
    .rate-container {
      background-color: #454545;
    }
  }

</style>
