<template>
  <div class="about">
    <h3 v-if="loading">Loading...</h3>
    <h4 v-for="link in links" :key="link.id">{{ link.description }}</h4>
    <input type="text" v-model="filter" />
  </div>
</template>
<script>
import gql from 'graphql-tag';
import get from 'lodash/get';
import Feed from '../graphql/Feed.gql';

export default {
  name: 'About',
  data() {
    return {
      filter: '',
    };
  },
  computed: {
    loading() {
      return this.$apollo.queries.feed.loading;
    },
    links() {
      return get(this.feed, 'links');
    },
  },
  apollo: {
    feed: {
      query: Feed,
      variables() {
        return { filter: this.filter };
      },
      subscribeToMore: {
        document: gql`
          subscription {
            newLink {
              id
              description
              url
            }
          }
        `,
        updateQuery: (prev, { subscriptionData }) => {
          if (!subscriptionData.data) return prev;
          const { newLink } = subscriptionData.data;
          const { __typename } = newLink;

          return Object.assign({}, prev, {
            feed: {
              links: [newLink, ...prev.feed.links],
              count: prev.feed.links.length + 1,
              __typename,
            },
          });
        },
      },
    },
    // $subscribe: {
    //   notice: {
    //     query: gql`
    //       subscription {
    //         newLink {
    //           id
    //           description
    //         }
    //       }
    //     `,
    //     result(data) {
    //       console.log('subscription funcionou!!!!!', data);
    //     },
    //   },
    // },
  },
  // async created() {
  //   const teste = await this.$apollo.query({
  //     query: Feed,
  //     variables() {
  //       return { filter: this.filter };
  //     },
  //   });
  // },
};
</script>
