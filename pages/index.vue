<template>
    <section>
        <div>
            <h1 class="line-one">Museum Art</h1>
            <h1 class="line-two">Showcase</h1>
        </div>
        <NuxtImg src="hero.jpg" />
    </section>
    <div class="container">
        <div v-for="(content, index) in dividedContents" :key="index" class="content-div">
            <div v-for="(item, i) in content" class="masonry-item imgDiv">
                <div class="overlay"></div>
                <NuxtImg v-if="item.primaryImageSmall" @click="originalImage = item.primaryImage; isShow = !isShow" :src="item.primaryImageSmall" loading="lazy" />
                <div class="details">
                    <p class="title">{{ item.title }}</p>
                    <p class="year">{{ item.objectDate }}</p>
                </div>
            </div>
        </div>
    </div>
    <button @click="loadMore">Show More</button>
    <div v-show="isShow" class="fullView" @click="isShow = !isShow">
        <div class="close"><svg xmlns="http://www.w3.org/2000/svg" width="1em" height="1em"
                viewBox="0 0 1024 1024">
                <path fill="currentColor"
                    d="M195.2 195.2a64 64 0 0 1 90.496 0L512 421.504L738.304 195.2a64 64 0 0 1 90.496 90.496L602.496 512L828.8 738.304a64 64 0 0 1-90.496 90.496L512 602.496L285.696 828.8a64 64 0 0 1-90.496-90.496L421.504 512L195.2 285.696a64 64 0 0 1 0-90.496" />
            </svg></div>
        <NuxtImg v-if="originalImage" :src="originalImage" loading="lazy" />
    </div>
</template>

<script setup>
import { useDebounceFn } from '@vueuse/core'
import { gsap } from 'gsap';
const objectsDetails = ref([]);
const isLoading = ref(false);
const isShow = ref(false);
const originalImage = ref(null);

const { data: objects, pending, error } = useAsyncData('department-objects-details', async () => {
    const response = await fetch(`https://collectionapi.metmuseum.org/public/collection/v1/objects?departmentIds=11`);
    const { objectIDs } = await response.json();
    return objectIDs;
});
const fetchObjectDetails = async (startCount, endCount) => {
    isLoading.value = true;
    loadMoreCount.value = endCount;
    const first20ObjectIDs = objects.value.slice(startCount, endCount);
    objectsDetails.value.push(...await Promise.all(
        first20ObjectIDs.map(objectID =>
            fetch(`https://collectionapi.metmuseum.org/public/collection/v1/objects/${objectID}`)
                .then(res => res.json()).finally()
        )
    ))
    console.log(objectsDetails.value)
    isLoading.value = false;
}
// const strings = ref(['Item 1', 'Item 2', 'Item 3', 'Item 4', 'Item 5']);
const numberOfDivs = ref(1);
const loadMoreCount = ref(null)

// Distribute strings across divs
const dividedContents = computed(() => {
    let result = Array.from({ length: numberOfDivs.value }, () => []);
    let imageIndex = 0;
    objectsDetails.value.forEach((item, index) => {
        if (item.primaryImageSmall) {
            const divIndex = index % numberOfDivs.value;
            result[divIndex].push(item);
            // imageIndex++;
        }
    });
    return result;
});

const updateDivCount = () => {
    if (window.innerWidth < 768) {
        numberOfDivs.value = 1;
    } else if (window.innerWidth >= 768 && window.innerWidth < 1024) {
        numberOfDivs.value = 2;
    } else {
        numberOfDivs.value = 4;
    }
};

// Scroll event handler with debouncing
const handleScroll = useDebounceFn(() => {
    const scrollPosition = window.scrollY + window.innerHeight;
    const triggerPoint = document.documentElement.scrollHeight * 0.6;

    if (scrollPosition >= triggerPoint && !isLoading.value) {
        loadMore();
    }
}, 10);

onMounted(async () => {
    window.addEventListener('scroll', handleScroll);
    window.addEventListener('resize', updateDivCount);
    await fetchObjectDetails(0, 20);
    await updateDivCount(); // Initial check
    await nextTick()
    gsap.to('.masonry-item', {
        opacity: 1,
        scale: 1,
        stagger: 0.1,
        duration: 0.1,
        ease: 'power3.out',
        from: { opacity: 0, scale: 0.5 }, // Starting from invisible and scaled down
    });
});

onUnmounted(() => {
    window.removeEventListener('scroll', handleScroll);
    window.removeEventListener('resize', updateDivCount);
});

watch(objectsDetails, updateDivCount); // Optionally re-calculate on strings change

const loadMore = async () => {
    await fetchObjectDetails(loadMoreCount.value, loadMoreCount.value + 20);
    await nextTick();
    gsap.to('.masonry-item', {
        opacity: 1,
        scale: 1,
        stagger: 0.1,
        duration: 0.1,
        ease: 'power3.out',
        from: { opacity: 0, scale: 0.5 }, // Starting from invisible and scaled down
    });
}
</script>


<style>
body {
    margin: 0;
}

.close {
    cursor: pointer;
    font-size: 25px;
    position: absolute;
    right: 10px;
    top: 10px;
    color: whitesmoke;
}

.close svg {
    pointer-events: none;
}
.fullView {
    position: fixed;
    width: 100%;
    height: 100%;
    top: 0;
    z-index: 5;
    background: #000000ab;
    backdrop-filter: blur(5px);
}

.fullView img {
    width: 100%;
    height: 100%;
    object-fit: contain;
    padding: 10px;
    box-sizing: border-box;
}

.line-one {
    color: #ffffffdb;
    bottom: 78px;
    line-height: 0.7;
    z-index: 2;
}

.line-two {
    color: #ffffffdb;
    z-index: 1;
}

section {
    width: 100%;
    height: 100vh;
}

section img {
    height: 100%;
    object-fit: cover;
}

section h1 {
    position: absolute;
    bottom: 0;
    font-size: 75px;
    margin: 0;
}

.overlay {
    pointer-events: none;
    display: none;
    width: 100%;
    height: 100%;
    position: absolute;
    background: linear-gradient(360deg, rgb(66, 66, 66) 0%, rgba(255, 255, 255, 0) 40%);
    margin: -10px;
}

.details {
    position: absolute;
    bottom: 0;
    padding: 10px;
    color: white;
    display: none;
}

.masonry-item:hover .details,
.masonry-item:hover .overlay {
    display: block;
}

.details p {
    margin: 0;
    padding: 0;
}

.details .title {
    font-size: 18px;
    font-weight: bold;
}

.details .year {
    font-size: 14px;
}

.masonry-item {
    opacity: 0;
    transform: scale(0.1);
    transition: transform 0.1s, opacity 0.1s;
    cursor: pointer;
}


.container {
    display: flex;
    justify-content: center;
}

.content-div {
    width: 100%;
    position: relative;
}

.div-container {
    display: flex;
}

img {
    width: 100%;
}

.imgDiv {
    padding: 10px;
}

/* Mobile styles */
.div-container>div {
    display: none;
}

.div-container>div:first-child {
    display: block;
}

/* Tablet styles */
@media (min-width: 768px) {
    .div-container>div {
        display: none;
    }

    .div-container>div:first-child,
    .div-container>div:nth-child(2) {
        display: block;
    }
}

/* Desktop styles */
@media (min-width: 1024px) {
    .div-container>div {
        display: block;
    }
}


@media (min-width: 769px) {
    section h1 {
        font-size: 135px;
    }

    .line-one {
        bottom: 125px;
        line-height: 0.7;
    }
}

@media (min-width: 1025px) {
    section h1 {
        font-size: 200px;
    }

    .line-one {
        bottom: 182px;
        line-height: 0.7;
    }
}
</style>