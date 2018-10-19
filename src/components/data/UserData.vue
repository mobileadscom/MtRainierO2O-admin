<template>
	<div>
		<div class="data-info shadow-1" :class="{white:white}">
			<div class="data-info-property">User Id: {{data.id}}</div>
			<div class="data-info-property">{{translateWord('registeredOn')}}: {{data.formattedDate}}</div>
			<div class="data-info-property" v-if="data.isTwitterId">{{translateWord('twitterName')}}: {{data.twitterName}}</div>
			<div class="data-info-property" :class="{reissued: data.reissued}">{{translateWord('couponCode')}}: <span v-html="data.couponLink"></span></div>
			<!-- <div class="data-info-property">{{translateWord('source')}}: {{translateWord(data.source)}}</div> -->
			<div class="data-info-property">{{translateWord('state')}}: {{translateWord(data.state)}}</div>
			<div class="data-info-property" v-if="data.formatMarkedDate">{{translateWord('dateMarked')}}: {{translateWord(data.formatMarkedDate)}}</div>
			<div class="data-info-property">{{translateWord('logic')}}: {{translateWord(data.logic)}}</div>
			<!-- <div class="secondary" v-if="canReissue(data)" @click="$emit('reissue', data)" style="margin: 4px 0px;">{{translateWord('reissueCoupon')}}</div> -->
			<div class="small-loader" v-if="data.reissuing" style="margin: 4px 0px;"></div>
			<div class="data-info-property" v-if="data.reissued" v-html="data.reissueResult"></div>
        </div>
	</div>
</template>

<script type="text/javascript">
	export default {
		name: 'UserData',
		props: ['data', 'lang', 'white'],
		data() {
			return {
				processedData: {},
				wording: {
					'registeredOn': {
						'en': 'Registered On',
						'ja': '登録日時'
					},
					'couponCode': {
						'en': 'Coupon Code',
						'ja': 'クーポンコード'
			        },
			        'source': {
						'en': 'Source',
						'ja': '店舗'
			        },
			        'state': {
						'en': 'State',
						'ja': '状態'
			        },
			        'reissueCoupon': {
						'en': 'Reissue Coupon',
						'ja': 'クーポン再発行'
			        },
			        'twitterName': {
						'en': 'Twitter Name',
						'ja': 'Twitter 名'
					},
					'dateMarked': {
						'en': 'Marked On',
						'ja': 'Marked On'
					},
					'logic': {
						'en': 'Logic',
						'ja': 'Logic'
					}
				}
			}
		},
		methods: {
			canReissue(val) {
				return val.state == 'win' && !val.reissued && !val.reissuing
			},
			translateWord(word) {
				if (this.wording[word]) {
					return this.wording[word][this.lang];
				}
				else {
					return word;
				}
		    }
		},
		mounted() {
			if (!this.lang) {
				this.lang = 'ja';
			}
		},
		watch: {

		}
	}
</script>

<style type="text/css">
	.data-info {
		margin: 5px 5px 15px;
	    padding: 4px 10px;
	    background-color: #fafafa;
	}

	.data-info.white {
		background-color: white;
	}

	.data-info-property {
		padding: 2px;
	    transition: all 0.3s;
	}

	.reissued {
	    background-color: #7fff00;
	}
</style>