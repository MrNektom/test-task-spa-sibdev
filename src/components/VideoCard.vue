<script setup>
    import { ref } from "@vue/reactivity";
    defineProps({
        video:Object
    })

    const search = ref("")

    const onClick = () => {
        location = "https://youtube.com/watch?v=" + __props.video.id.videoId
    }
</script>
<template>
    <div class="video-card" @click="onClick">
        <el-image class="video-card__thumbnail" fit="cover" :src="video.snippet.thumbnails.medium.url" alt=""/>
        <div class="video-card__info">
            <span class="video-card__title" v-html="video.snippet.title"></span>
            <span class="video-card__description" v-html="video.snippet.description"></span>
        </div>
    </div>
</template>
<style>
    .video-card {
        border-radius: 5px;
        box-shadow: 0 0 5px #0004;
        overflow: hidden;
    }

    .video-card__thumbnail {
        width: 100%;
        max-width: 100%;
        max-height: 140px;
    }


    .video-card .video-card__title {
        display: block;
        font-size: 14px;
        font-weight: 500;
        white-space: nowrap;
        text-overflow: ellipsis;
        max-width: 100%;
        padding: 0px 10px;
        overflow: hidden;

    }
    .video-card__description {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        font-size: 14px;
        color: #555;
        white-space: nowrap;
        margin: 5px 10px;
    }
    .video-cards.list_mode .video-card__thumbnail {
        height: 100%;
        width: 170px;
        min-width: 170px;
        max-width: 170px;
    }

    .video-cards.list_mode .video-card {
        display: flex;
        height: 88px;
    }
</style>