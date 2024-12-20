<template>
    <div>
        <Header />
        <div class="container mx-auto p-4">
            <div class="slider-wrapper">
                <slick-carousel :autoplay="true" :autoplay-speed="3000" :dots="true">
                    <div v-for="(item, index) in slides" :key="index">
                        <img :src="item.image" alt="slide" class="w-full h-64 object-cover" />
                    </div>
                </slick-carousel>
            </div>

            <h2
                class="text-2xl font-bold text-gray-800 mt-16"
                v-if="productsByPublisher.length > 0 && productsByPublisher[0].MaNXB"
            >
                Tất cả sách
            </h2>

            <Carousel
                class="relative w-full mt-2"
                :opts="{
                    align: 'start',
                }"
            >
                <CarouselContent>
                    <CarouselItem
                        v-for="product in productsByPublisher"
                        :key="product._id"
                        class="basis-1/5 flex-shrink-0 relative overflow-hidden transition-transform duration-300 ease-in-out rounded-lg shadow-lg group hover:shadow-xl hover:-translate-y-2"
                    >
                        <router-link v-if="product.slug" :to="`/bookDetails/${product.slug}`">
                            <div class="p-1">
                                <Card>
                                    <CardContent class="p-4">
                                        <img
                                            :src="product.HinhAnhSach[0]"
                                            alt="product"
                                            class="w-full h-48 object-cover rounded-t-lg"
                                        />
                                        <h3 class="mt-4 text-xl font-semibold line-clamp-2 custom-min-height">
                                            {{ product.TenSach }}
                                        </h3>

                                        <div class="mt-4 flex justify-between items-center">
                                            <p class="text-gray-500 whitespace-normal dark:text-gray-400">
                                                {{ product.TacGia }}
                                            </p>
                                            <span class="text-lg font-medium text-red-600 dark:text-white">{{
                                                formatCurrency(product.DonGia | currency)
                                            }}</span>
                                        </div>
                                        <div class="mt-4 flex justify-between items-center">
                                            <Heart class="cursor-pointer" />
                                            <ShoppingCart class="cursor-pointer" />
                                        </div>
                                    </CardContent>
                                </Card>
                            </div>
                        </router-link>
                    </CarouselItem>
                </CarouselContent>
                <CarouselPrevious />
                <CarouselNext />
            </Carousel>
        </div>
        <Footer />
    </div>
</template>

<script lang="ts" setup>
import Header from '@/components/Header.vue';
import Footer from '@/components/Footer.vue';

import axios from '@/utils/axios';

import { ref, onMounted, watch } from 'vue';
import { Card, CardContent } from '@/components/ui/card';
import { Carousel, CarouselContent, CarouselItem, CarouselNext, CarouselPrevious } from '@/components/ui/carousel';
import { Button } from '@/components/ui/button';

import { Heart, ShoppingCart } from 'lucide-vue-next';
const productsByPublisher = ref([]);

const fetchProductsByPublisher = async () => {
    try {
        const res = await axios.get('/book/publisher/66c459aedad7de9f848ba263');
        productsByPublisher.value = res.data.products;
        console.log('productsByPublisher.value: ', productsByPublisher.value);
    } catch (error) {
        console.error('Error fetching home:', error);
    }
};

const fetchProductsByAllPublishers = async () => {
    try {
        const res = await axios.get('/publisher/getAllPublishers');
        const publishers = res.data.publishers;

        const productPromises = publishers.map((publisher) => {
            console.log('publisher14242: ', publisher);
            return axios.get(`/book/publisher/${publisher._id}`).then((productRes) => {
                console.log('productRes.data.products21212: ', productRes.data.products);
                return productRes.data.products;
            });
        });

        const allProducts = await Promise.all(productPromises);
        console.log('allProducts: ', allProducts);
        console.log('allProducts.flat(): ', allProducts.flat());
        productsByPublisher.value = allProducts.flat();
        console.log('All products by all publishers: ', productsByPublisher.value);
    } catch (error) {
        console.error('Error fetching publisher:', error.response ? error.response.data : error.message);
        if (error.response) {
            console.log('Error status:', error.response.status);
            console.log('Error data:', error.response.data);
        }
    }
};

const formatCurrency = (value: number) => {
    return new Intl.NumberFormat('vi-VN', { style: 'currency', currency: 'VND' }).format(value);
};
onMounted(() => {
    // fetchProductsByPublisher();
    fetchProductsByAllPublishers();
});
</script>

<style scoped>
.line-clamp-2 {
    display: -webkit-box;
    -webkit-line-clamp: 2; /* Hiển thị tối đa 2 dòng */
    -webkit-box-orient: vertical;
    overflow: hidden; /* Ẩn nội dung dư */
    text-overflow: ellipsis; /* Thêm dấu "..." */
}
.custom-min-height {
    min-height: 56px;
}
</style>
