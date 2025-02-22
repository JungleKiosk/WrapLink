<script>
import ColorMode from "./ColorMode.vue";


export default {
    name: "Navbar",
    components: {
        ColorMode,
    }
};
</script>

<template>
    <nav class="navbar justify-content-center">
        <ul>
            <li><router-link to="/">Home</router-link></li>
            <!-- <li><router-link to="/data">Data</router-link></li> -->
            <li>
                <ColorMode />
            </li>
        </ul>
    </nav>
</template>

<style scoped>
.navbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
}
</style>