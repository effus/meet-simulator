<template>
    <v-content class="py-0">
        <v-row v-if="botMessage || nextItem || caseItem">
            <v-col>
                <v-card v-if="botMessage" class="mx-auto bot-message-container" outlined>
                    <v-card-title>Message editor</v-card-title>
                    <v-card-text>
                        <v-textarea
                            label="This is what bot says to user"
                            no-resize
                            outlined
                            rows="5"
                            v-model="tempText"
                            ></v-textarea>
                    </v-card-text>
                </v-card>
                
                <v-card v-if="nextItem" class="mx-auto py-0" outlined>
                    <v-card-title>Next message condition</v-card-title>
                    <v-card-text>
                        <v-container class="py-0">
                            <v-row>
                                <v-col class="body-1 pt-1 pb-0">
                                    If user points greater than
                                </v-col>
                                <v-col cols="12" class="py-0 next-slider-container">
                                    <v-slider
                                        dense
                                        :disabled="flagFreezePoints"
                                        v-model="tempPoints"
                                        color="blue darken-4"
                                        min="1"
                                        max="100"
                                        step="1"
                                        :label="String(tempPoints)"
                                    ></v-slider>
                                </v-col>
                            </v-row>
                            <v-row>
                                <v-col class="body-1 pt-2 pb-0">
                                    Bot will say this message
                                </v-col>
                                <v-col cols="12" class="py-0 next-combobox-container">
                                    <v-combobox label="Target message ID" v-model="tempComboId" :items="comboMessageList"></v-combobox>
                                </v-col>
                            </v-row>
                            
                        </v-container>
                        
                    </v-card-text>
                </v-card>

                <v-card v-if="caseItem" class="mx-auto py-0" outlined>
                    <v-card-title class="py-1">Answer case</v-card-title>
                    <v-card-text>
                        <v-container class="py-0">
                            <v-row>
                                <v-col class="body-1 pt-1 pb-0">
                                    Points for answer
                                </v-col>
                                <v-col cols="12" class="py-0 points-slider-container">
                                    <v-slider
                                        dense
                                        v-model="tempPoints"
                                        color="blue darken-4"
                                        min="-10"
                                        max="10"
                                        step="1"
                                        :label="String(tempPoints)"
                                    ></v-slider>
                                </v-col>
                            </v-row>
                            <v-row>
                                <v-col class="body-1 pt-2 pb-0">
                                    User message
                                </v-col>
                                <v-col cols="12" class="py-0 answer-text-container">
                                    <v-textarea
                                        outlined
                                        no-resize
                                        rows="2"
                                        v-model="tempText"
                                        ></v-textarea>
                                </v-col>
                            </v-row>
                            
                        </v-container>
                        
                    </v-card-text>
                </v-card>
            </v-col>
        </v-row>

        <v-row>
            <v-col>
                <v-bottom-navigation v-if="botMessage">
                    <v-btn @click="onClickApplyBotMessage" class="primary"><span>Apply</span><v-icon>mdi-check</v-icon></v-btn>
                    <v-btn @click="onClickCancelBotMessage" class="secondary"><span>Cancel</span><v-icon>mdi-cancel</v-icon></v-btn>
                </v-bottom-navigation>  

                <v-bottom-navigation v-if="nextItem">
                    <v-btn @click="onClickApplyNextItem" class="primary"><span>Apply</span><v-icon>mdi-check</v-icon></v-btn>
                    <v-btn @click="onClickCancelNextItem" class="secondary"><span>Cancel</span><v-icon>mdi-cancel</v-icon></v-btn>
                    <v-btn v-if="nextItem.points > 0" @click="onClickDeleteNextItem" class="secondary"><span>Delete</span><v-icon>mdi-server-remove</v-icon></v-btn>
                </v-bottom-navigation>

                <v-bottom-navigation v-if="caseItem">
                    <v-btn @click="onClickApplyCaseItem" class="primary"><span>Apply</span><v-icon>mdi-check</v-icon></v-btn>
                    <v-btn @click="onClickCancelCaseItem" class="secondary"><span>Cancel</span><v-icon>mdi-cancel</v-icon></v-btn>
                    <v-btn @click="onClickDeleteCaseItem" class="secondary"><span>Delete</span><v-icon>mdi-server-remove</v-icon></v-btn>
                </v-bottom-navigation>

                <v-bottom-navigation v-else>

                    <v-btn class="px-0" @click="onClickNewMessage"><span>New</span><v-icon>mdi-beaker-plus-outline</v-icon></v-btn>

                    <template v-if="currentId">
                        <v-btn v-if="!isFirst" class="px-0" @click="onClickDeleteMessage"><span>Delete</span><v-icon>mdi-beaker-remove-outline</v-icon></v-btn>
                        <v-btn class="px-0" @click="onClickAddCondition"><span>Add condition</span><v-icon>mdi-help-network-outline</v-icon></v-btn>
                        <v-btn class="px-0" @click="onClickAddCase"><span>Add case</span><v-icon>mdi-graph</v-icon></v-btn>
                        <v-btn class="px-0 save-btn" @click="onClickSave"><span>Save</span><v-icon>mdi-cloud-upload</v-icon></v-btn>
                    </template>

                </v-bottom-navigation>  

            </v-col>
        </v-row>

  </v-content>

</template>

<script>

const RESERVED_WORD_FINISH = ':finish:';
const ReservedWords = [
    RESERVED_WORD_FINISH
];

export default {
    props: {
        currentId: {
            type: Number,
            default: null
        },
        botMessage: {
            type: String,
            default: () => ''
        },
        nextItem: {
            type: Object,
            default: () => ({
                id: -1,
                points: 0,
                goto: null,
                list: []
            })
        },
        caseItem: {
            type: Object,
            default: () => ({
                id: -1,
                points: 0,
                text: null
            })
        },
        isFirst: {
            type: Boolean,
            default: false
        }
    },
    data() {
        return {
            tempText: '',
            tempPoints: 0,
            flagFreezePoints: false,
            tempComboId: null
        }
    },
    methods: {
        // default actions
        onClickNewMessage() {
            this.$emit('new-message');
        },
        onClickDeleteMessage() {
            this.$emit('delete-message');
        },
        onClickAddCondition() {
            this.$emit('add-condition');
        },
        onClickAddCase() {
            this.$emit('add-case');
        },
        // bot message actions
        onClickApplyBotMessage() {
            const text = this.tempText;
            this.tempText = '';
            if (!text) {
                throw Error('Empty message text');
            }
            this.$emit('apply-bot-message', {
                text: text,
            });
        },
        onClickCancelBotMessage() {
            this.tempText = '';
            this.$emit('cancel-bot-message');
        },
        // next item actions
        onClickApplyNextItem() {
            if (!this.tempComboId) {
                throw Error('Empty value for selected');
            }
            let gotoValue = this.tempComboId.replace(/#/,'');
            const intValue = parseInt(gotoValue);
            if (!gotoValue) {
                throw Error('Value not found');
            }
            if (isNaN(intValue) && !ReservedWords.includes(gotoValue)) {
                throw Error('Bad value for target');
            } else if (!isNaN(intValue)) {
                gotoValue = String(intValue);
            }
            if (this.nextItem.points === 0) {
                this.tempPoints = 0;
            }
            this.$emit('apply-next-item', {
                id: this.nextItem.id,
                points: this.tempPoints,
                goto: gotoValue
            });
        },
        onClickCancelNextItem() {
            this.$emit('cancel-next-item');
        },
        onClickDeleteNextItem() {
            this.$emit('delete-next-item', {
                id: this.nextItem.id,
            });
        },
        onClickApplyCaseItem() {
            if (!this.tempText) {
                throw Error('Message is empty');
            }
            this.$emit('apply-case-item', {
                id: this.caseItem.id,
                points: this.tempPoints,
                text: this.tempText
            });
        },
        onClickCancelCaseItem() {
            this.$emit('cancel-case-item');
        },
        onClickDeleteCaseItem() {
            this.$emit('delete-case-item', {
                id: this.caseItem.id,
            });
        },
        onClickSave() {
            this.$emit('save');
        }
    },
    watch: {
        botMessage(value) {
            this.tempText = value;
        },
        "nextItem.id": function(value) {
            if (value) {
                this.tempPoints = this.nextItem.points;
                this.tempComboId = this.nextItem.goto ? '#' + this.nextItem.goto : null;
                this.flagFreezePoints = !this.nextItem.points;
            } else {
                this.tempPoints = 0;
                this.tempComboId = null;
                this.flagFreezePoints = true;
            }
        },
        "caseItem.id": function(value) {
            console.debug('caseItem.id', value);
            if (value) {
                this.tempPoints = this.caseItem.points;
                this.tempText = this.caseItem.text;
            } else {
                this.tempPoints = 0;
                this.tempText = '';
            }
        }
    },
    computed: {
        comboMessageList() {
            let list = this.nextItem.list.filter(value => {
                if (value.id !== this.currentId) {
                    return true;
                }
            });
            list.push(RESERVED_WORD_FINISH);
            return list;
        }
    }
}
</script>

<style lang="scss">
.v-bottom-navigation {
    .v-btn__content {
        white-space: normal;
        width: 100%;
    }

}
</style>