// calender.vue

<template>
<div class="ds-dropdown" ref='container_dom'>
	<span href="#"
		v-for="week in weeks"
		:key="week.id"
		:class="{ 'ds-dropdown-highlight': week.current }"
		:ref="(week.current) ? 'current_week_dom' : null"
		@click="$emit('nav', week.moment)"
	>{{ week.moment.startOf('week').format('l') + ' - ' + week.moment.endOf('week').format('l') }}</span>
</div>
</template>

<style scoped>
.ds-dropdown {
	display: block;
	position: absolute;
	background-color: #f1f1f1;
	width: 100%;
	box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.2);
	z-index: 1;
	margin-top: 2px;
	height: 700%; /* vertically span 7 tabs */
	min-height: 100px;
	overflow-y: scroll;
	overflow-x: hidden;
}

.ds-dropdown span {
	display: block;
	padding: 5px 0;
	border-collapse: collapse;
	border-spacing: 0px;
	outline: 1px solid #ddd;
	box-sizing: border-box;
	background-color: white;
	font-weight: normal;
	text-decoration: none;
	-webkit-user-select: none;
	-khtml-user-select: none;
	-moz-user-select: -moz-none;
	-o-user-select: none;
	user-select: none;
}

.ds-dropdown span:hover {
	cursor: pointer;
	background-color: #f1f1f1;

}

span.ds-dropdown-highlight {
	background-color: #fffacd87;
}

.ds-dropdown-hide {
	display: none;
}
</style>

<script>

export default {
	created: function() {
		var self = this;
		this.cur_week = moment(this.today).startOf('week');
		if (!this.nav_week)
			this.nav_week = this.cur_week;

		// generate N past/future weeks
		const N = 10;
		let past = spanNumWeeks.call(this, this.cur_week, -N);
		let future = spanNumWeeks.call(this, this.cur_week, N);
		this.weeks = past.concat(future);

		// register
		this.$nextTick(function() {
			// register clicks
			let dom = self.$refs.container_dom;
			if (dom && dom.previousElementSibling) {
				dom.previousElementSibling.addEventListener('click', function(evt) {
					dom.classList.toggle('ds-dropdown-hide');
					self.scrollToNavWeek();
					evt.preventDefault();
					evt.stopPropagation();
				});
			}
			window.addEventListener('click', function(evt) {
				dom.classList.add('ds-dropdown-hide');
			});

			// scroll to the week on display
			this.scrollToNavWeek();
			dom.classList.add('ds-dropdown-hide');

			// register scrolls
			let mute = false;
			setTimeout(function() {
				dom.addEventListener('scroll', function(evt) {
					if (mute) {
						return;
					}
					if (dom.scrollTop <= 0) {
						// scroll to top
						let m = self.weeks[0].moment;
						let past = spanNumWeeks.call(self, m, -N);
						self.weeks.splice(0, 0, ...past);
						muteTemporarily();
						scrollToNthWeek.call(self, N);
					} else if (dom.scrollTop >= dom.scrollHeight-dom.offsetHeight) {
						// scroll to bot
						let m = moment(self.weeks[self.weeks.length-1].moment)
								.add(1, 'week');
						let future = spanNumWeeks.call(self, m, N);
						self.weeks.splice(self.weeks.length, 0, ...future);
						muteTemporarily();
						scrollToNthWeek.call(self, self.weeks.length-1-N);
					}

					// disable event listener for a short while
					function muteTemporarily() {
						if (mute == true) { return; }
						mute = true;
						setTimeout(() => {
							mute = false;
						}, 10);
					}

					// Wait for render finishes, then scroll to nth item of
					// the dropdown menu
					function scrollToNthWeek(nth) {
						let self = this;
						this.$nextTick(function() {
							self.$refs
								.container_dom
								.querySelector(`span:nth-child(${nth})`)
								.scrollIntoView(true);
						});
					}

				}); // end of scroll
			}, 10); // end of setTimeout()
		}); // end of register

		// Construct array of week objects from @from_week
		// to forward/backward @n weeks.
		function spanNumWeeks(from_week, n) {
			let self = this;
			from_week = moment(from_week);
			if (n < 0) {
				from_week.add(n, 'week');
				n = -n;
			}

			return new Array(n).fill(undefined).map((val, i) => {
				let m = moment(from_week).add(i, 'week').startOf('week');
				return {
					id: m.format('x'),
					moment: m,
					current: m.isSame(self.cur_week, 'week'),
				};
			});
		}
	},

	props: {
		// moment() of today, use to highlight current week item
		today: {
			type: Object,
			default: moment(),
		},
		// the week you are currently navigating in
		nav_week: {
			type: Object,
			default: null,
		},
	},

	data () {
		return {
			weeks: null,
		}
	},

	methods: {
		/**
		 * Scroll to current week
		 */
		scrollToCurrentWeek() {
			this.$refs
				.cur_week_dom[0]
				.scrollIntoView(true);
		},

		/**
		 * Scroll to the week you are current navigating/editing
		 * TODO: Add items to dropdown menu if there is no items
		 *       for `this.nav_week`
		 */
		scrollToNavWeek() {
			var cur_week_dom = this.$refs.current_week_dom[0];
			var d = moment.duration(this.nav_week.diff(this.cur_week));
			d = parseInt(Math.ceil(d.asWeeks())); // num of weeks to current week
			var dom = getNthElementSibling(cur_week_dom, d);
			dom.scrollIntoView(true);

			// get the forward/backward n-th sibiling element
			function getNthElementSibling(from_dom, n) {
				if ((!from_dom) || (n == 0)) {
					return from_dom;
				} else {
					let forward = (n > 0);
					let next_dom = (forward) ? from_dom.nextElementSibling : from_dom.previousElementSibling;
					return getNthElementSibling(next_dom, (forward) ? n-1 : n+1);
				}
			}
		}
	},
}
</script>
