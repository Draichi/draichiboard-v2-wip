<template>
  <q-page class="flex flex-center">
    <q-card v-if="!loading" style="width: 100vw;">
      <BarChart :data="data"/>
    </q-card>
  </q-page>
</template>

<script>
import BarChart from 'components/BarChart';

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
    await this.getContributions('1ccb8080cf2512553979aad93dd621961093b2be', 'Draichi');
  },
  methods: {
    async getContributions(token, username) {
      const headers = {
        Authorization: `bearer ${token}`,
      };
      const body = {
        query: `query {
            user(login: "${username}") {
              contributionsCollection(from: "2020-04-10T00:00:00", to: "2020-06-04T00:00:00") {
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
      // console.log(dataaa);
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
      console.log(contributionCalendar.weeks);
      const callbackForData = (item) => item.contributionDays.map((i) => i.contributionCount);
      const callbackForLabel = (item) => item.contributionDays.map((i) => i.date);
      const data = contributionCalendar.weeks.map(callbackForData).flat();
      const labels = contributionCalendar.weeks.map(callbackForLabel).flat();
      console.log(data);
      console.log(labels);

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
