<template>
  <div v-show="show" class="home">
    <!-- by Apple Music 模块-->
    <div
      v-if="settings.showPlaylistsByAppleMusic !== false"
      class="index-row first-row"
    >
      <div class="title"> by Apple Music</div>
      <CoverRow
        :type="'playlist'"
        :items="byAppleMusic"
        sub-text="appleMusic"
        :image-size="1024"
      />
    </div>
    <!-- 推荐歌单 模块 -->
    <div class="index-row">
      <div class="title">
        {{ $t('home.recommendPlaylist') }}
        <router-link to="/explore?category=推荐歌单"
          >{{ $t('home.seeMore') }}
        </router-link>
      </div>
      <CoverRow
        :type="'playlist'"
        :items="recommendPlaylist.items"
        sub-text="copywriter"
      />
    </div>
    <!-- for you 模块 -->
    <div class="index-row">
      <div class="title"> For You</div>
      <div class="for-you-row">
        <!-- 每日推荐模块 -->
        <DailyTracksCard ref="DailyTracksCard" />
        <FMCard />
      </div>
    </div>
    <!-- 推荐艺人 模块 -->
    <div class="index-row">
      <div class="title">{{ $t('home.recommendArtist') }}</div>
      <CoverRow
        type="artist"
        :column-number="6"
        :items="recommendArtists.items"
      />
    </div>
    <!-- 新专速递 模块 -->
    <div class="index-row">
      <div class="title">
        {{ $t('home.newAlbum') }}
        <router-link to="/new-album">{{ $t('home.seeMore') }}</router-link>
      </div>
      <CoverRow
        type="album"
        :items="newReleasesAlbum.items"
        sub-text="artist"
      />
    </div>
    <!-- 排行榜 模块 -->
    <div class="index-row">
      <div class="title">
        {{ $t('home.charts') }}
        <router-link to="/explore?category=排行榜"
          >{{ $t('home.seeMore') }}
        </router-link>
      </div>
      <CoverRow
        type="playlist"
        :items="topList.items"
        sub-text="updateFrequency"
        :image-size="1024"
      />
    </div>
  </div>
</template>

<script>
import { toplists } from '@/api/playlist';
import { toplistOfArtists } from '@/api/artist';
import { newAlbums } from '@/api/album';
import { byAppleMusic } from '@/utils/staticData';
import { getRecommendPlayList } from '@/utils/playList';
import NProgress from 'nprogress';
import { mapState } from 'vuex';
import CoverRow from '@/components/CoverRow.vue';
import FMCard from '@/components/FMCard.vue';
import DailyTracksCard from '@/components/DailyTracksCard.vue';

export default {
  name: 'Home',
  components: { CoverRow, FMCard, DailyTracksCard },
  data() {
    return {
      show: false,
      recommendPlaylist: { items: [] },
      newReleasesAlbum: { items: [] },
      topList: {
        items: [],
        ids: [19723756, 180106, 60198, 3812895, 60131],
      },
      recommendArtists: {
        items: [],
        indexs: [],
      },
    };
  },
  computed: {
    ...mapState(['settings']),
    byAppleMusic() {
      return byAppleMusic;
    },
  },
  activated() {
    this.loadData();
    this.$parent.$refs.scrollbar.restorePosition();
  },
  methods: {
    loadData() {
      // 设置进度调，将原先的 1000 调整带 10
      setTimeout(() => {
        if (!this.show) NProgress.start();
      }, 10);

      getRecommendPlayList(10, false).then(items => {
        this.recommendPlaylist.items = items;
        NProgress.done();
        this.show = true;
      });

      newAlbums({
        area: this.settings.musicLanguage ?? 'ALL',
        limit: 10,
      }).then(data => {
        this.newReleasesAlbum.items = data.albums;
      });

      const toplistOfArtistsAreaTable = {
        all: null,
        zh: 1,
        ea: 2,
        jp: 4,
        kr: 3,
      };
      toplistOfArtists(
        toplistOfArtistsAreaTable[this.settings.musicLanguage ?? 'all']
      ).then(data => {
        let indexs = [];
        // 随机插入 6 首歌
        while (indexs.length < 6) {
          let tmp = ~~(Math.random() * 100);
          if (!indexs.includes(tmp)) indexs.push(tmp);
        }
        this.recommendArtists.indexs = indexs;
        this.recommendArtists.items = data.list.artists.filter((l, index) =>
          indexs.includes(index)
        );
      });
      toplists().then(data => {
        this.topList.items = data.list.filter(l =>
          this.topList.ids.includes(l.id)
        );
      });
      this.$refs.DailyTracksCard.loadDailyTracks();
    },
  },
};
</script>

<style lang="scss" scoped>
.index-row {
  margin-top: 54px;
}

.index-row.first-row {
  margin-top: 32px;
}

.playlists {
  display: flex;
  flex-wrap: wrap;
  margin: {
    right: -12px;
    left: -12px;
  }

  .index-playlist {
    margin: 12px 12px 24px 12px;
  }
}

.title {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-bottom: 20px;
  font-size: 28px;
  font-weight: 700;
  color: var(--color-text);

  a {
    font-size: 13px;
    font-weight: 600;
    opacity: 0.68;
  }
}

footer {
  display: flex;
  justify-content: center;
  margin-top: 48px;
}

.for-you-row {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 24px;
  margin-bottom: 78px;
}
</style>
