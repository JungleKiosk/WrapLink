<template>
    <div class="theme-toggle">
        <button @click="toggleTheme" class="theme-button rounded-5">
            {{ isDarkMode ? 'LMode' : 'DMode' }}
        </button>
    </div>
</template>

<script>
export default {
    name: "ThemeToggle",
    data() {
        return {
            isDarkMode: false,
        };
    },
    mounted() {
        const currentTheme = localStorage.getItem("theme");
        if (currentTheme === "dark-mode") {
            this.isDarkMode = true;
            document.body.classList.add("dark-mode");
        } else {
            document.body.classList.add("light-mode");
        }
    },
    methods: {
        toggleTheme() {
            this.isDarkMode = !this.isDarkMode;
            if (this.isDarkMode) {
                document.body.classList.remove("light-mode");
                document.body.classList.add("dark-mode");
                localStorage.setItem("theme", "dark-mode");
            } else {
                document.body.classList.remove("dark-mode");
                document.body.classList.add("light-mode");
                localStorage.setItem("theme", "light-mode");
            }
        },
    },
};
</script>

<style scoped></style>