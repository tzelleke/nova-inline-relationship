<template>
    <div>
        <div v-for="(field, attrib) in fields"
            :key="attrib"
            class="nova-items-field-input-wrapper w-full">
            <component :is="'form-' + field.component"
                :ref="attrib"
                :field="field"
                :full-width-content="true"
                :errors="errors"
                :resource-id="modelId"
                :resource-name="modelKey">
            </component>
        </div>
    </div>
</template>

<script>
    export default {
        name: "relationship-form-item",

        props: [
            'value',
            'id',
            'modelId',
            'modelKey',
            'errors',
            'field'
        ],

        computed: {
            fields() {
                return _.keyBy(
                    Object.keys({ ...this.value }).map(
                        attrib => {
                            return {
                                ...{
                                    'options': {}
                                },
                                ...this.value[attrib].meta,
                                ...{
                                    'attribute': attrib,
                                    'attribute_': (this.value[attrib].meta.component === "file-field") ?
                                        attrib + '?' + this.id :
                                        this.field.attribute + '_' + this.id + '_' + attrib, // This is needed to enable delete link for file without triggering duplicate id warning
                                    'name': this.value[attrib].meta.singularLabel,
                                    'deletable': this.modelId > 0, // Hide delete button if model Id is not present, i.e. new model
                                    'attrib': attrib
                                }
                            }
                        }
                    ), 'attrib'
                )
            },

            label() {
                return this.field.singular
                    ? this.field.singularLabel
                    : `${this.field.singularLabel} ${this.id + 1}`;
            }
        },

        methods: {
            getValueFromChildren() {
                return _.tap(new FormData(), formData => {
                    _(this.$refs).each(item => {
                        if (item[0].field.component === 'file-field') {
                            if (item[0].file) {
                                formData.append(item[0].field.attrib, item[0].file, item[0].fileName);
                            } else if (item[0].value) {
                                formData.append(item[0].field.attrib, String(item[0].value))
                            }
                        } else if (item[0].field.component === 'boolean-field') {
                            formData.append(item[0].field.attribute_, item[0].trueValue);
                        } else {
                            item[0].fill(formData);
                        }
                    })
                })
            },

            fill(formData, parentAttrib) {
            	formData.append(`${parentAttrib}[${this.id}][modelId]`, this.modelId);
                this.getValueFromChildren().forEach(
                    (value, key) => {
                        formData.append(`${parentAttrib}[${this.id}][values][${key}]`, value);
                    }
                );
            },

            removeItem() {
                this.$emit('deleted', this.id);
            },
        },
    }
</script>
