<template>
    <div>
        <h1>HOME</h1>
        <router-link to="/test"> TestPage </router-link>
        <button @click.prevent="tiggerEndpoint">Trigger Endpoint</button>
        <p v-if="response">{{ response.data }}</p>
    </div>
</template>
