<template>
  <q-page class="flex flex-center">
    <q-card v-if="!loading" style="width: 100vw;">
      <BarChart :data="data"/>
    </q-card>
  </q-page>
</template>

<script>
import BarChart from 'components/BarChart';
import { githubToken } from '../../.env.local.js';

export default {
  name: 'PageIndex',
  components: {
    BarChart,
  },
  data() {
    return {
      data: null,
      loading: true,
    };
  },
  async mounted() {
    await this.getContributions(githubToken, 'Draichi');
  },
  methods: {
    async getContributions(token, username) {
      const headers = {
        Authorization: `bearer ${token}`,
      };
      const body = {
        query: `query {
            user(login: "${username}") {
              contributionsCollection(from: "2020-01-10T00:00:00", to: "2020-06-04T00:00:00") {
                totalIssueContributions
                totalPullRequestContributions
                totalPullRequestReviewContributions
                totalRepositoriesWithContributedCommits
                totalRepositoriesWithContributedIssues
                totalRepositoriesWithContributedPullRequests
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
        // totalIssueContributions,
        // totalPullRequestContributions,
        // totalPullRequestReviewContributions,
        // totalRepositoriesWithContributedCommits,
        // totalRepositoriesWithContributedIssues,
        contributionCalendar,
      } = dataaa.data.user.contributionsCollection;
      // console.log(totalIssueContributions,
      //   totalPullRequestContributions,
      //   totalPullRequestReviewContributions,
      //   totalRepositoriesWithContributedCommits,
      //   totalRepositoriesWithContributedIssues);
      // console.log(contributionCalendar.totalContributions);
      // console.log(contributionCalendar.weeks);
      const callbackForData = (item) => {
        // console.log(item.contributionDays);
        const weekContributions = item.contributionDays
          .reduce((acc, val) => acc.concat(val.contributionCount), []);
        // const weekContributionsClean = weekContributions.reduce((a, b) => a + b, 0);
        // console.log(
        //   weekContributionsClean,
        // );
        return weekContributions.reduce((a, b) => a + b, 0);
      };
      const monthNames = ['January', 'February', 'March', 'April', 'May', 'June',
        'July', 'August', 'September', 'October', 'November', 'December',
      ];
      const callbackForLabel = (item) => {
        const { length } = item.contributionDays;
        const { date } = item.contributionDays[length - 1];
        const monthNumber = new Date(date).getMonth();
        const formatedDate = monthNames[monthNumber];
        return formatedDate;
        // return item.contributionDays.map((i) => i.date);
      };
      const rawData = contributionCalendar.weeks.map(callbackForData).flat();
      const chunkArray = (array, size) => {
        if (array.length <= size) {
          return [array];
        }
        return [array.slice(0, size), ...chunkArray(array.slice(size), size)];
      };
      const dataByMonth = chunkArray(rawData, 4);
      const data = dataByMonth.map((item) => item.reduce((a, b) => a + b));
      const rawLabels = contributionCalendar.weeks.map(callbackForLabel).flat();
      const labels = [...new Set(rawLabels)];
      this.data = {
        labels,
        datasets: [{
          label: 'Custom Label Name',
          // backgroundColor: gradient,
          pointBackgroundColor: 'white',
          borderWidth: 1,
          borderColor: '#911215',
          data,
        }],
      };
      this.loading = false;
    },
  },
};
</script>
