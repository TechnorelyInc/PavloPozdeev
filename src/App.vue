<template>
  <div class="image-finder">
    <form
      class="form"
      @submit.prevent="getImages"
    >
      <input
        v-model="query"
        class="input"
        placeholder="Search images"
        type="text"
      />

      <button class="search-button" type="submit">
        Search
      </button>
    </form>

    <div v-if="loading" class="title">
      Loading...
    </div>

    <div v-if="isErrorMessage" class="title">
       Something went wrong...
    </div>

    <div v-if="isMessage && !isErrorMessage" class="title">
       There are no images...
    </div>

    <div class="image-container">
      <a
        v-for="(image, index) in images"
        :key="index"
        :href="image.link"
        target="_blank"
      >
        <img
          :alt="image.alt"
          :src="image.src"
        />
      </a>
    </div>
  </div>
</template>

<script lang="ts">
import axios from 'axios';

type TImage = {
  src: string,
  alt: string,
  link: string,
};

type TUnslpashImage = {
  urls: {
    regular: string,
  },
  'alt_description': string,
  links: {
    html: string,
  }
};

type TPexelsImage = {
  src: {
    medium: string,
  },
  photographer: string,
  url: string,
};

type TPixabayImage = {
  webformatURL: string,
  user: string,
  pageURL: string,
};

type TData = {
  query: string,
  images: TImage[],
  page: number,
  loading: boolean,
  isErrorMessage: boolean,
  isMessage: boolean,
};

export default {
  layout: 'articles',
  data(): TData {
    return {
      query: '',
      images: [],
      page: 1,
      loading: false,
      isErrorMessage: false,
      isMessage: false,
    }
  },
  mounted() {
    window.addEventListener('scroll', this.handleScroll);
  },
  unmounted() {
    window.removeEventListener('scroll', this.handleScroll);
  },
  methods: {
    async doQuery(): Promise<void> {
      if (!this.query.trim()) {
        this.query = '';
        return;
      }
      this.loading = true;
      this.isMessage = false;
      this.isErrorMessage = false;

      try {
        const unsplashResponse = await axios.get(`https://api.unsplash.com/search/photos?{this.page}&query=${this.query}&per_page=20&client_id=${import.meta.env.VITE_UNSPLASH_API_KEY}`);

        const pexelsResponse = await axios.get(`https://api.pexels.com/v1/search?page=${this.page}&query=${this.query}&per_page=20`, {
          headers: {
            Authorization: `${import.meta.env.VITE_PEXELS_API_KEY}`,
          }
        });

        const pixabayResponse = await axios.get(`https://pixabay.com/api/?key=${import.meta.env.VITE_PIXABEY_API_KEY}&q=${this.query}&image_type=photo&page=${this.page}&per_page=20`);

        const unsplashImages = unsplashResponse.data.results.map((item: TUnslpashImage): TImage => ({
          src: item.urls.regular,
          alt: item.alt_description,
          link: item.links.html,
        }));
        const pexelsImages = pexelsResponse.data.photos.map((item: TPexelsImage): TImage => ({
          src: item.src.medium,
          alt: item.photographer,
          link: item.url,
        }));
        const pixabayImages = pixabayResponse.data.hits.map((item: TPixabayImage): TImage => ({
          src: item.webformatURL,
          alt: item.user,
          link: item.pageURL
        }));

        this.images = [
          ...this.images,
          ...unsplashImages,
          ...pexelsImages,
          ...pixabayImages
        ];

        this.page += 1;
      } catch (error) {
        console.error(error);
        this.isErrorMessage = true;
      } finally {
        this.loading = false;

        if (!this.images.length) {
          this.isMessage = true;
        }
      }
    },
    getImages(): void {
      this.page = 1;
      this.images = [];
      this.doQuery();
    },
    handleScroll(): void {
      const scrollPosition = window.innerHeight + window.pageYOffset + 500;
      const pageHeight = document.documentElement.scrollHeight;

      if (scrollPosition >= pageHeight && !this.loading) {
        this.doQuery();
      }
    },
  }
};
</script>

<style lang="scss">
* {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}

.image-finder {
  margin: 0 auto;
  max-width: 1600px;
  min-height: 100vh;
  padding: 0 50px 20px;
}

.form {
  align-items: center;
  background-color: #fff;
  display: flex;
  height: 80px;
  justify-content: center;
  left: 0;
  position: sticky;
  right: 0;
  top: 0;
}

.input {
  border-bottom-left-radius: 5px;
  border-right: 0;
  border-top-left-radius: 5px;
  font-size: 18px;
  font-weight: 500;
  height: 40px;
  min-width: 400px;
  outline: transparent;
  padding-left: 10px;
}

.search-button {
  background-color: #fff;
  border-bottom-right-radius: 5px;
  border-top-right-radius: 5px;
  cursor: pointer;
  font-weight: 600;
  height: 40px;
  transition: color, 0.3s, background-color, 0.3s;
  padding: 0 5px;
}

.title {
  font-size: 50px;
  font-weight: 700;
  margin: 50px 0;
  text-align: center;
}

.search-button:hover {
  background-color: #5656f0;
  color: #fff;
}

.image-container {
  display: grid;
  gap: 5px;
  grid-template-columns: repeat(auto-fill, 280px);
  grid-template-rows: repeat(auto-fill, 250px);
  justify-content: center;
}

.image-container img {
  cursor: pointer;
  height: 250px;
  width: 100%;
}
</style>
