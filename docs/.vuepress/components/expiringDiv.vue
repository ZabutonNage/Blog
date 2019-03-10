<template>
    <div v-if="!expired">
        <slot></slot>
    </div>
</template>

<script>
    export default {
        name: `expiring-div`,
        props: {
            start: {
                type: String,
                validator: val => /20[12]\d-(0[1-9]|1[012])-(0[1-9]|[12]\d|3[01])/.test(val) && !Number.isNaN(Date.parse(val))
            },
            days: Number
        },
        data() {
            return {
                expired: Date.now() > Date.parse(this.start) + (this.days * 24 * 60 * 60 * 1000)
            };
        }
    }
</script>
