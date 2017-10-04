<template>
    <div class="dropdown">
        <div v-on:click="toggleDropdown()" class="dropbtn">{{ resultDate }}</div>
        <div id="calendar-dropdown" class="dropdown-content" @mouseenter="lockCalendarScroll" @mouseleave="unlockCalendarScroll" @wheel="roller($event)">
            <div class="calendar-header">
                <div class="cal-header-title">{{getMonthFormatted}}.{{year}}</div>
                <div class="cal-header-buttons">
                    <span class="icon-arrow-up btn" @click="prevMonth()"></span>
                    <span class="icon-arrow-down btn" @click="nextMonth()"></span>
                </div>
            </div>
            <div class="calendar-daysOfWeek">
                <span v-for="item in weeks" v-text="item"></span>
            </div>
            <div class="calendar-content">
                <template v-for="(item,index) in dates">
                    <div class="elem prev-next-month" v-if="isOutOfMonth(index)">{{item}}</div>
                    <div v-else @click="selectDate(item)" class="elem available" :class="{selected: isSelected(item), today: isToday(item)}">
                        {{item}}
                    </div>
                </template>
            </div>
        </div>
    </div>
</template>

<script>
    import * as moment from 'moment'

    const CELL_COUNT = 7 * 6

    export default {
        props: {
            dateFormat: {
                type: String
            },
            value: {
                type: Date,
                default: null
            }
        },
        data () {
            return {
                dates: [],
                year: 0,
                month: 0,
                resultDate: 'Due date',
                scrollTop: '',

                firstDay: 0,
                lastDay: 0,

                weeks: []
            }
        },
        computed: {
            date () {
                return this.value ? new Date(this.value) : new Date()
            },
            getMonthFormatted () {
                return this.month < 10 ? '0' + this.month : this.month
            }
        },
        methods: {
            toggleDropdown () {
                document.getElementById('calendar-dropdown').classList.toggle('show')
            },
            isOutOfMonth (index) {
                return index < this.firstDay - 1 ||
                    index > this.lastDay + this.firstDay - 2
            },
            prevMonth () {
                this.date.setMonth(this.date.getMonth() - 1)
                this.generateMonth()
            },
            isSelected (dayDate) {
                if (this.value) {
                    return this.year === this.value.getFullYear() &&
                        this.month === (this.value.getMonth() + 1) &&
                        dayDate === this.value.getDate()
                } else {
                    return false
                }
            },
            isToday (dayDate) {
                return this.year === (new Date()).getFullYear() &&
                    this.month === ((new Date()).getMonth() + 1) &&
                    dayDate === (new Date()).getDate()
            },
            nextMonth () {
                this.date.setMonth(this.date.getMonth() + 1)
                this.generateMonth()
            },
            generateMonth () {
                const dates = []

                this.year = this.date.getFullYear()
                const month = this.date.getMonth()

                const firstDay = new Date(this.year, month, 1).getDay()
                const lastDate = new Date(this.year, month + 1, 0).getDate()

                // fill this month
                const startIndex = (firstDay + 5) % 7
                let i = 1
                let count = startIndex
                while (count++ < lastDate + startIndex) {
                    dates[count] = i++
                }

                // fill previous month
                let prevLastDate = new Date(this.year, month, 0).getDate()
                let prevCount = firstDay === 1 ? 7 : (firstDay + 6) % 7 // monday starts on the 2nd line
                i = prevCount
                while (i--) {
                    dates[i] = prevLastDate--
                }

                // fill next month
                const nextIndex = lastDate + prevCount
                let nextCount = CELL_COUNT - nextIndex
                i = 0
                while (nextCount--) {
                    dates[nextIndex + i] = ++i
                }

                this.dates = dates
                this.month = month + 1
                this.firstDay = firstDay < 2 ? firstDay + 7 : firstDay
                this.lastDay = lastDate
            },
            selectDate (dayDate) {
                let value = new Date(this.date.getFullYear(), this.date.getMonth(), dayDate)
                this.$emit('input', value)
                this.toggleDropdown()
            },
            lockCalendarScroll () {
                this.scrollTop = window.scrollY
                document.body.style.top = '-' + this.scrollTop + 'px'
                document.body.classList.add('overflowHidden')
            },
            unlockCalendarScroll () {
                document.body.classList.remove('overflowHidden')
                document.body.style.top = '0px'
                window.scrollTo(0, this.scrollTop)
            },
            roller (e) {
                if (e.deltaY < 0) {
                    this.prevMonth()
                } else {
                    this.nextMonth()
                }
            },
            generateDayOfWeekSymbols () {
                let weeks = new Array(7)
                let m = moment()
                for (let i = 0; i < 7; i++) {
                    weeks[(m.day() + 6) % 7] = m.format('dddd').substr(0, 1)
                    m.add(1, 'days')
                }
                for (let i = 0; i < 7; i++) {
                    this.weeks.push(weeks[i])
                }
            }
        },
        mounted () {
            this.generateDayOfWeekSymbols()
            this.date.setTime((new Date()).getTime())
            this.generateMonth()
        },
        watch: {
            value: function (newValue) {
                this.resultDate = newValue ? moment(newValue).format(this.dateFormat).toString() : 'Due date'
            }
        }
    }
</script>

<style lang="scss">

    @import "../../assets/common";

    $elementPx: 26px;
    $elementMargin: 2px;
    $contentWidth: 210px;
    $dropdownPadding: 4px;
    $dropdownWidth: 218px;

    .dropbtn {
        border: none;
        cursor: pointer;
        display: flex;
        width: 100%;
        height: 100%;
        align-items: center;
        color: $greyDarker;
    }

    .dropdown {
        position: relative;
        color: $greyDarkest;
        font-size: 12px;
        line-height: 14px;
        background: transparent;
        height: 4em;
        width: 100%;
        display: inline-block;
    }

    .show {
        display: inline-block !important;
    }
    .overflowHidden {
        position: fixed;
        overflow-y: scroll;
        width: 100%;
    }

    .calendar-panel {
        width: 360px;
        height: 420px;
        top: 40px;
        color: #333;
        border-radius: 2px;
        border: 1px solid #D3DCE6;
        box-shadow: 0 2px 6px #CCC;
        user-select: none;
    }

    .dropdown-content {
        display: none;
        position: absolute;
        background-color: #f9f9f9;
        width: $dropdownWidth;
        z-index: 5;
        left: 0;
        border-radius: 4px;
        padding: $dropdownPadding;
        overflow: auto;
        box-shadow: 0 8px 16px 0 rgba(0,0,0,0.2);
        right: 0;
        cursor: default;
    }

    .calendar-header {
        height: 30px;
        display: flex;

        .cal-header-title {
            width: 140px;
            margin-left: 10px;
            align-items: center;
            display: flex;
        }
        .cal-header-buttons {
            width: 60px;
            font-family: KM-Iconfont;
            justify-content: center;
            align-items: center;
            display: flex;

            .btn {
                padding: 0 5px;
                margin: 0 2px;
                cursor: pointer;
            }
        }
    }
    .calendar-daysOfWeek {
        display: flex;
        justify-content: space-between;
        color: $greyDarkest;
        width: $contentWidth;

        & span {
            display: block;
            width: 45px;
            line-height: 36px;
            text-align: center;
            font-weight: 500;
        }
    }

    .calendar-content {
        display: flex;
        justify-content: space-between;
        flex-wrap: wrap;
        align-items: center;
        width: $contentWidth;
        height: 100%;

        .elem {
            display: flex;
            color: inherit;
            line-height: 36px;
            width: $elementPx;
            height: $elementPx;
            border-radius: 100%;
            margin: $elementMargin;
            justify-content: center;
            align-items: center;
        }

        & .available {
            cursor: pointer;
        }
        & .available:hover {
            background-color: $accentLighter;
        }

        & .prev-next-month {
            color: #DDD;
            cursor: default;
        }

        & .today {
            color: $accent;
        }

        & .selected {
            color: white;
            background-color: $accent;
        }

        & .selected:hover {
            background-color: $accent;
        }
    }

</style>
