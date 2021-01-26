<template>
    <div class="library">
        <p class="library__name">{{this.name}}</p>
        <p class="library__description">{{this.description}}</p>
        <div class="library__contacts">
            <p><span>Телефон:</span> {{this.phone ? `+${this.phone}` : 'отсутствует'}}</p>
            <p><span>Email:</span> {{this.email ? this.email : 'отсутствует'}}</p>
        </div>
    </div>
</template>


<script>
    import axios from "axios";

    export default {
        data() {
            return {
                description: '',
                name: '',
                email: '',
                phone: ''
            }
        },
        async created() {
            try {
                fetch('/config.json').then(res => res.json()).then( async config => {
                    this.confs = config;
                    const res = await axios.get(`${this.confs.hostUrl}_id=` + this.$route.params.id)
                    this.description = res.data[0].data.general.description.replaceAll('<p>', '').replaceAll('</p>', '').replaceAll('<span>', '').replaceAll('</span>', '').replaceAll('<ul>', '').replaceAll('</ul>', '').replaceAll('<li>', '').replaceAll('</li>', '')
                    this.name = res.data[0].data.general.name
                    res.data[0].data.general.contacts.phones.length ? this.phone = res.data[0].data.general.contacts.phones[0].value : ''
                    res.data[0].data.general.contacts.email ? this.email = res.data[0].data.general.contacts.email : ''
                })
            } catch(e) {
                console.error(e)
            }
        }
    }
</script>


<style scoped lang="scss">
    .library{
        padding: 20px 70px;

        &__name{
            font-size: 24px;
            font-weight: bold;
            text-align: center;
        }

        &__description{
            font-size: 19px;
        }

        &__contacts{
            font-size: 19px;

            span{
                font-weight: bold;
            }
        }
    }

    @media (max-width: 500px) {
        .library{
            padding: 20px;
        }
    }

</style>
