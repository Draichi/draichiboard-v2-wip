<template>
    <div class="row">
      <div class="col-auto column">
        <q-card v-if="!loading" class="q-ma-md">
          <BarChart :data="dataPie" />
          <BarChart :data="dataPie" />
        </q-card>
        <q-card v-else>Loading</q-card>

      </div>
      <div class="col">
        <q-card v-if="!loading" style="width: 60vw;" class="q-ma-md">
          <LineChart :data="sanitazedData" />
        </q-card>
        <q-card v-else>Loading</q-card>

        <q-card v-if="!loading" class="q-ma-md">
          <LineChart :data="sanitazedData" />
        </q-card>
        <q-card v-else>Loading</q-card>
      </div>
    </div>
</template>

<script>
import LineChart from 'components/LineChart';
import BarChart from 'components/BarChart';
import { githubToken } from '../../.env.local.js';

const today = new Date().toISOString();

const getWeeklyContributions = (week) => {
  const weekContributionsArray = week.contributionDays.reduce(
    (acc, val) => acc.concat(val.contributionCount),
    [],
  );
  const weekContributions = weekContributionsArray.reduce((a, b) => a + b, 0);
  return weekContributions;
};

const monthNames = [
  'Jan',
  'Feb',
  'Mar',
  'Apr',
  'May',
  'Jun',
  'Jul',
  'Aug',
  'Sep',
  'Oct',
  'Nov',
  'Dec',
];

const getContributionsByMonth = (array, size) => {
  if (array.length <= size) {
    return [array];
  }
  return [
    array.slice(0, size),
    ...getContributionsByMonth(array.slice(size), size),
  ];
};

const sanitazeData = (contributionCalendarWeeks) => {
  const weeklyContributions = contributionCalendarWeeks
    .map(getWeeklyContributions)
    .flat();
  const montlyContributionsArray = getContributionsByMonth(
    weeklyContributions,
    4,
  );
  const montlyContributions = montlyContributionsArray.map((item) => item.reduce((a, b) => a + b));
  return montlyContributions;
};

const sanitazeLabels = (contributionCalendarWeeks, yearLabel) => {
  const weeklyContributions = contributionCalendarWeeks
    .map((item) => {
      const { length } = item.contributionDays;
      const { date } = item.contributionDays[length - 1];
      const monthNumber = new Date(date).getMonth();
      const formatedDate = `${monthNames[monthNumber]} ${yearLabel}`;
      return formatedDate;
    })
    .flat();
  const montlyContributionsArray = getContributionsByMonth(
    weeklyContributions,
    4,
  );
  const montlyContributions = montlyContributionsArray.map(
    (item) => item[item.length - 1],
  );
  return montlyContributions;
};

export default {
  name: 'PageIndex',
  components: {
    LineChart,
    BarChart,
  },
  data() {
    return {
      sanitazedData: null,
      data: [],
      labels: [],
      loading: true,
      dataPie: null,
      totalIssueContributions: null,
      totalPullRequestContributions: null,
      totalCommitContributions: null,
      totalPullRequestReviewContributions: null,
      totalRepositoriesWithContributedCommits: null,
      totalRepositoriesWithContributedIssues: null,
      totalRepositoryContributions: null,
      years: [
        {
          from: '2020-01-01T00:00:00',
          to: today,
          label: '20',
        },
        {
          from: '2019-01-01T00:00:00',
          to: '2019-12-01T00:00:00',
          label: '19',
        },
        {
          from: '2018-01-01T00:00:00',
          to: '2018-12-01T00:00:00',
          label: '18',
        },
        {
          from: '2017-11-01T00:00:00',
          to: '2017-12-01T00:00:00',
          label: '17',
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

      const response = await fetch('https://api.github.com/graphql', {
        method: 'POST',
        body: JSON.stringify(body),
        headers,
      });
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
      this.totalIssueContributions = totalIssueContributions;
      this.totalPullRequestContributions = totalPullRequestContributions;
      this.totalCommitContributions = totalCommitContributions;
      this.totalPullRequestReviewContributions = totalPullRequestReviewContributions;
      this.totalRepositoriesWithContributedCommits = totalRepositoriesWithContributedCommits;
      this.totalRepositoriesWithContributedIssues = totalRepositoriesWithContributedIssues;
      this.totalRepositoryContributions = totalRepositoryContributions;
      const contributionCalendarWeeks = contributionCalendar.weeks;
      // > this data is yearly so this is just the last fetched year
      this.dataPie = {
        // labels: this.labels,
        labels: [
          'totalRepositoryContributions',
          'totalRepositoriesWithContributedCommits',
          'totalCommitContributions',
        ],
        datasets: [
          {
            // label: 'Total contributions',
            backgroundColor: [
              'rgba(120, 74, 211, 0.29)',
              'rgba(74, 211, 150, 0.29)',
            ],
            // pointBackgroundColor: 'white',
            // borderWidth: 1,
            // borderColor: '#911215',
            // data: this.data,
            data: [
              totalRepositoryContributions,
              totalRepositoriesWithContributedCommits,
              totalCommitContributions,
            ],
          },
        ],
      };
      // this.data.push(...sanitazeData(contributionCalendarWeeks));
      // this.labels.push(...sanitazeLabels(contributionCalendarWeeks, year.label));
      return {
        data: await sanitazeData(contributionCalendarWeeks),
        label: await sanitazeLabels(contributionCalendarWeeks, year.label),
      };
    },
    async getYearlyContributions() {
      const data20 = await this.getContributions(
        githubToken,
        'Draichi',
        this.years[0],
      );
      const data19 = await this.getContributions(
        githubToken,
        'Draichi',
        this.years[1],
      );
      const data18 = await this.getContributions(
        githubToken,
        'Draichi',
        this.years[2],
      );
      const data17 = await this.getContributions(
        githubToken,
        'Draichi',
        this.years[3],
      );
      // this.years.forEach((year) => {
      //   const responsta = this.getContributions(githubToken, 'Draichi', year);
      //   datinha[year.label] = responsta;
      //   this.datinha.push(datinha[year.label]);
      // });

      this.sanitazedData = {
        // labels: this.labels,
        labels: [
          ...data17.label,
          ...data18.label,
          ...data19.label,
          ...data20.label,
        ],
        datasets: [
          {
            label: 'Total contributions',
            // backgroundColor: gradient,
            pointBackgroundColor: 'white',
            borderWidth: 1,
            borderColor: '#911215',
            // data: this.data,
            data: [
              ...data17.data,
              ...data18.data,
              ...data19.data,
              ...data20.data,
            ],
          },
        ],
      };
      this.loading = false;
    },
  },
};
</script>

<style lang="scss">
  div>span#text-rotate:before{
    content: '1';
    font-size: 2rem;
    animation-name: head;
    animation-duration: 5s;
    animation-iteration-count: infinite;
}

@keyframes head {
    0%  {font-size: 2rem; opacity:1;}
    20% {font-size: 1rem; opacity:0;}
    40% {font-size: 2rem; opacity:1; content: "2"}
    60% {font-size: 1rem; opacity:0;}
    80% {font-size: 2rem; opacity:1; content: '3'}
    90% {font-size: 1rem; opacity:0;}
    100% {font-size: 2rem;opacity:1;}
}
</style>
