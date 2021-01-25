<template>
    <nav>
        <div class="nav">
            <router-link to="/" class="nav__link">Список библиотек</router-link>
        </div>
    </nav>
</template>

<script>
    export default {
        name: "Navbar"
    }
</script>

<style scoped lang="scss">
    .nav {
        height: 70px;
        background-color: #1d1d1f;
        width: 100%;
        font-size: 25px;
        padding-top: 18px;

        &__link {
            text-decoration: none;
            color: rgb(245, 245, 247);
            margin-left: 70px;
            font-weight: 400;
            font-style: normal;
        }
    }

    @media (max-width: 500px) {
        .nav {
            &__link {
                margin-left: 20px;
            }
        }
    }
</style>
