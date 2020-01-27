<template>
    <div class="container">
        <h1>Event Scheduler</h1>
        <div class="row">
            <div class="col-md-4">
                <form class="">
                    <div class="form-group">
                        <label for="event" class="">Event</label>
                        <input name="event" id="event" type="text" class="form-control" v-model="event" v-bind:class="{ 'is-invalid': errors.hasOwnProperty('event') }">
                        <div class="invalid-feedback" v-if="errors.hasOwnProperty('event')">{{errors.event[0]}}</div>
                    </div>
                    <div class="form-group" v-bind:class="{ 'is-invalid': errors.hasOwnProperty('start_date') }">
                        <label class="">Start Date</label>
                        <date-picker v-model="start_date" valueType="format"></date-picker>
                        <div class="invalid-feedback" v-if="errors.hasOwnProperty('start_date')">{{ errors.start_date[0] }}</div>
                    </div>
                    <div class="form-group" v-bind:class="{ 'is-invalid': errors.hasOwnProperty('end_date') }">
                        <label class="">End Date</label>
                        <date-picker v-model="end_date" valueType="format" :disabled="start_date == null ? true : false"></date-picker>
                        <div class="invalid-feedback" v-if="errors.hasOwnProperty('end_date')">{{ errors.end_date[0] }}</div>
                    </div>
                    <div class="form-check">
                        <label class="form-check-label">
                        <input type="checkbox" class="form-check-input" v-model="days_of_the_weeks"  :disabled="end_date == null ? true : false" value="monday"> Monday</label>
                    </div>
                    <div class="form-check">
                        <label class="form-check-label">
                        <input type="checkbox" class="form-check-input" v-model="days_of_the_weeks"  :disabled="end_date == null ? true : false" value="tuesday"> Tuesday</label>
                    </div>
                    <div class="form-check">
                        <label class="form-check-label">
                        <input type="checkbox" class="form-check-input" v-model="days_of_the_weeks"  :disabled="end_date == null ? true : false" value="wednesday"> Wednesday</label>
                    </div>
                    <div class="form-check">
                        <label class="form-check-label">
                        <input type="checkbox" class="form-check-input" v-model="days_of_the_weeks"  :disabled="end_date == null ? true : false" value="thursday"> Thursday</label>
                    </div>
                    <div class="form-check">
                        <label class="form-check-label">
                        <input type="checkbox" class="form-check-input" v-model="days_of_the_weeks"  :disabled="end_date == null ? true : false" value="friday"> Friday</label>
                    </div>
                    <div class="form-check">
                        <label class="form-check-label">
                        <input type="checkbox" class="form-check-input" v-model="days_of_the_weeks"  :disabled="end_date == null ? true : false" value="saturday"> Saturday</label>
                    </div>
                    <div class="form-check">
                        <label class="form-check-label">
                        <input type="checkbox" class="form-check-input" v-model="days_of_the_weeks" :disabled="end_date == null ? true : false" value="sunday"> Sunday</label>
                    </div>
                    <br>
                    <button type="button" class="btn btn-success" v-on:click="submit">Submit</button>
                    <button type="button" class="btn btn-danger">Cancel</button>
                </form>
            </div>
            <div class="col-md-8">
                <div class="demo-app">
                    <div className='demo-app-calendar' style="position:relative;">
                        <button v-on:click="previousMonth" style="position:absolute; left:0;">Prev</button>
                        <button v-on:click="nextMonth" style="position:absolute; right:0;">Next</button>
                        <FullCalendar
                            defaultView="dayGridMonth"
                            ref="fullCalendar"
                            :plugins="calendarPlugins"
                            :events="calendarEvents"
                            :header="calendarHeader"
                        />
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import DatePicker from 'vue2-datepicker';
    import FullCalendar from '@fullcalendar/vue';
    import dayGridPlugin from '@fullcalendar/daygrid';
    import moment from 'moment';
    import Toast from "vue-toastification";

    Vue.use(Toast, {});

    export default {
        components: {
            FullCalendar, // make the <FullCalendar> tag available,
            DatePicker
        },
        mounted() {
            this.loadData();
        },
        data() {
            return {
                calendarHeader:{
                    left: '',
                    center: 'title',
                    right: ''
                },
                calendarPlugins: [ dayGridPlugin ],
                calendarEvents: [],
                defaultMonthStartDate: moment().startOf('month').format('YYYY-MM-DD'),
                defaultMonthEndDate: moment().endOf('month').format('YYYY-MM-DD'),
                event: '',
                start_date: null,
                end_date: null,
                days_of_the_weeks:[],
                errors: {},
            }
        },
        methods: {
            loadData: function () {
                axios({
                    method: 'GET',
                    url: `api/events?start_date=${this.defaultMonthStartDate}&end_date=${this.defaultMonthEndDate}`
                }).then(
                    (response) => {
                        this.calendarEvents = response.data.data;
                    }
                );
            },
            previousMonth: function () {
                let calendarApi = this.$refs.fullCalendar.getApi()
                calendarApi.prev();
                this.setCalendarDateRange(calendarApi.getDate());
            },
            nextMonth: function () {
                let calendarApi = this.$refs.fullCalendar.getApi()
                calendarApi.next();
                this.setCalendarDateRange(calendarApi.getDate());
            },
            setCalendarDateRange: function (currentMonthDate) {
                this.defaultMonthStartDate = moment(currentMonthDate).format('YYYY-MM-DD');
                this.defaultMonthEndDate = moment(currentMonthDate).endOf('month').format('YYYY-MM-DD');
                this.loadData();
            },
            isInvalidDate: function (string_date) {
                let class_date = 'mx-input';
                if(this.errors.hasOwnProperty(string_date)){
                    class_date += ' is-invalid';
                }
                return class_date;
            },
            clearForm: function () {
                this.event = '';
                this.start_date = null;
                this.end_date = null;
                this.days_of_the_weeks = [];
                this.errors = [];
            },
            submit: function () {
                this.errors = [];
                let formData = {
                    event: this.event,
                    start_date: this.start_date,
                    end_date: this.end_date,
                    days_of_the_weeks: this.days_of_the_weeks
                };

                if(moment(formData.start_date).isValid()){
                    formData.start_date = moment(formData.start_date).format('YYYY-MM-DD');
                }

                if(moment(formData.end_date).isValid()){
                    formData.end_date = moment(formData.end_date).format('YYYY-MM-DD');
                }

                axios({
                    method: 'POST',
                    url: `api/events`,
                    data: formData,
                    validateStatus: (status) => {
                        return true;
                    }
                })
                    .then(
                        (response) => {
                            if(response.status === 200) {
                                this.$toast.success(response.data.message, {});
                                this.clearForm();
                                this.loadData();
                            }
                            if(response.status === 422) {
                                this.errors = response.data.errors;
                            }
                        }
                    );
            }
        }
    }
</script>
