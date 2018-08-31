<template>
    <default-field :field="field">
        <template slot="field">
            <multiselect
                v-model="value"
                :options="options"
                :placeholder="this.field.indexName + ' ' +__('Select')"
                :selectLabel="__('Press enter to select')"
                :selectedLabel="__('Selected')"
                :deselectLabel="__('Press enter to remove')"
                :custom-label="customLabel"
                @input="onChange">
                <span slot="noResult">
                    {{ __('Oops! No elements found. Consider changing the search query.')}}
                </span>
            </multiselect>
            <p v-if="hasError" class="my-2 text-danger">
                {{ firstError }}
            </p>
        </template>
    </default-field>
</template>

<script>
import { FormField, HandlesValidationErrors } from "laravel-nova";
import Multiselect from "vue-multiselect";

export default {
    components: { Multiselect },
    mixins: [FormField, HandlesValidationErrors],

    props: ["resourceName", "resourceId", "field"],

    data() {
        return {
            options: []
        };
    },
    created() {
        console.log(this.field);

        if (this.field.dependsOn) {
            Nova.$on("nova-belongsto-depend-" + this.field.dependsOn, async dependsOnValue => {
                this.value = "";

                Nova.$emit("nova-belongsto-depend-" + this.field.attribute, {
                    value: this.value,
                    field: this.field
                });

                if (dependsOnValue && dependsOnValue.value) {
                    this.options = (await Nova.request().post("/nova-vendor/nova-belongsto-depend", {
                        resourceClass: this.field.resourceParentClass,
                        modelClass: dependsOnValue.field.modelClass,
                        attribute: this.field.attribute,
                        dependKey: dependsOnValue.value.id
                    })).data;

                    if (this.field.valueKey) {
                        this.value = this.options.find(item => item[this.field.modelKeyName] == this.field.valueKey);
                        Nova.$emit("nova-belongsto-depend-" + this.field.attribute, {
                            value: this.value,
                            field: this.field
                        });
                    }
                }
            });
        }
    },

    methods: {
        customLabel(item) {
            return item[this.field.titleKey];
        },
        /*
         * Set the initial, internal value for the field.
         */
        setInitialValue() {
            this.options = this.field.options;
            if (this.field.value) {
                this.value = this.options.find(item => item[this.field.modelKeyName] == this.field.valueKey);

                if (this.value) {
                    Nova.$emit("nova-belongsto-depend-" + this.field.attribute, {
                        value: this.value,
                        field: this.field
                    });
                }
            }
        },

        /**
         * Fill the given FormData object with the field's internal value.
         */
        fill(formData) {
            if (this.value) {
                formData.append(this.field.attribute, this.value.id || "");
            }
        },

        /**
         * Update the field's internal value.
         */
        handleChange(value) {
            this.value = value;
        },

        async onChange(value) {
            Nova.$emit("nova-belongsto-depend-" + this.field.attribute, {
                value,
                field: this.field
            });
        }
    }
};
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>