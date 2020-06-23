<template>
  <q-page class="flex flex-center">
    <q-card v-if="!loading" style="width: 100vw;">
      <LineChart :data="sanitazedData"/>
    </q-card>

    <!-- <q-card v-if="!loading" style="width: 100vw;">
      <BarChart :data="data"/>
    </q-card> -->
  </q-page>
</template>

<script>
import LineChart from 'components/LineChart';
// import BarChart from 'components/BarChart';
import { githubToken } from '../../.env.local.js';

const today = new Date().toISOString();

const getWeeklyContributions = (week) => {
  const weekContributionsArray = week.contributionDays
    .reduce((acc, val) => acc.concat(val.contributionCount), []);
  const weekContributions = weekContributionsArray.reduce((a, b) => a + b, 0);
  return weekContributions;
};

const monthNames = ['January', 'February', 'March', 'April', 'May', 'June',
  'July', 'August', 'September', 'October', 'November', 'December',
];

// const callbackForLabel = (item) => {
//   const { length } = item.contributionDays;
//   const { date } = item.contributionDays[length - 1];
//   const monthNumber = new Date(date).getMonth();
//   const formatedDate = `${monthNames[monthNumber]} / 20`;
//   return formatedDate;
// };

const getContributionsByMonth = (array, size) => {
  if (array.length <= size) {
    return [array];
  }
  return [array.slice(0, size), ...getContributionsByMonth(array.slice(size), size)];
};

const sanitazeData = (contributionCalendarWeeks) => {
  const weeklyContributions = contributionCalendarWeeks.map(getWeeklyContributions).flat();
  const montlyContributionsArray = getContributionsByMonth(weeklyContributions, 4);
  const montlyContributions = montlyContributionsArray.map((item) => item.reduce((a, b) => a + b));
  return montlyContributions;
};

const sanitazeLabels = (contributionCalendarWeeks, yearLabel) => {
  // const weeklyContributions = contributionCalendarWeeks.map(callbackForLabel).flat();
  const weeklyContributions = contributionCalendarWeeks.map((item) => {
    const { length } = item.contributionDays;
    const { date } = item.contributionDays[length - 1];
    const monthNumber = new Date(date).getMonth();
    const formatedDate = `${monthNames[monthNumber]} / ${yearLabel}`;
    return formatedDate;
  }).flat();
  const labels = [...new Set(weeklyContributions)];
  return labels;
};

export default {
  name: 'PageIndex',
  components: {
    LineChart,
    // BarChart,
  },
  data() {
    return {
      sanitazedData: null,
      data: [],
      labels: [],
      loading: true,
      years: [
        {
          from: '2020-01-01T00:00:00',
          to: today,
          label: '2020',
        },
        {
          from: '2019-01-01T00:00:00',
          to: '2019-12-01T00:00:00',
          label: '2019',
        },
        {
          from: '2018-01-01T00:00:00',
          to: '2018-12-01T00:00:00',
          label: '2018',
        },
        {
          from: '2017-01-01T00:00:00',
          to: '2017-12-01T00:00:00',
          label: '2017',
        },
      ],
    };
  },
  mounted() {
    this.getYearlyContributions();
  },
  methods: {
    async getContributions(token, username, year) {
      const headers = {
        Authorization: `bearer ${token}`,
      };
      const body = {
        // https://developer.github.com/v4/object/contributionscollection/
        query: `query {
            user(login: "${username}") {
              contributionsCollection(from: "${year.from}", to: "${year.to}") {
                totalCommitContributions
                totalIssueContributions
                totalPullRequestContributions
                totalPullRequestReviewContributions
                totalRepositoriesWithContributedCommits
                totalRepositoriesWithContributedIssues
                totalRepositoriesWithContributedPullRequests
                totalRepositoriesWithContributedPullRequestReviews
                totalRepositoryContributions
                contributionCalendar {
                  totalContributions
                  weeks {
                    contributionDays {
                      contributionCount
                      date
                    }
                  }
                }
              }
            }
          }`,
      };

      const response = await fetch('https://api.github.com/graphql', { method: 'POST', body: JSON.stringify(body), headers });
      const dataaa = await response.json();
      const {
        totalIssueContributions,
        totalPullRequestContributions,
        totalCommitContributions,
        totalPullRequestReviewContributions,
        totalRepositoriesWithContributedCommits,
        totalRepositoriesWithContributedIssues,
        totalRepositoryContributions,
        contributionCalendar,
      } = dataaa.data.user.contributionsCollection;
      console.log({
        totalIssueContributions,
        totalCommitContributions,
        totalRepositoryContributions,
        totalPullRequestContributions,
        totalPullRequestReviewContributions,
        totalRepositoriesWithContributedCommits,
        totalRepositoriesWithContributedIssues,
      });
      const contributionCalendarWeeks = contributionCalendar.weeks;
      this.data.push(...sanitazeData(contributionCalendarWeeks));
      this.labels.push(...sanitazeLabels(contributionCalendarWeeks, year.label));
      this.sanitazedData = {
        labels: this.labels,
        datasets: [{
          label: 'Custom Label Name',
          // backgroundColor: gradient,
          pointBackgroundColor: 'white',
          borderWidth: 1,
          borderColor: '#911215',
          data: this.data,
        }],
      };
      this.loading = false;
    },
    getYearlyContributions() {
      this.years.forEach((year) => {
        this.getContributions(githubToken, 'Draichi', year);
      });
    },
  },
};
</script>
