<template>
    <div class="code-mirror-single-line"></div>
</template>

<script>
import { EditorView, keymap, placeholder } from '@codemirror/view'
import { EditorState } from '@codemirror/state'
import { history, historyKeymap } from '@codemirror/commands'

function createState(vueInstance) {
    return EditorState.create({
        doc: vueInstance.modelValue,
        extensions: [
            history(),
            EditorView.updateListener.of(v => {
                if(v.docChanged) {
                    vueInstance.emitted = true
                    vueInstance.$emit('update:modelValue', v.state.doc.toString())
                }
            }),
            // From: https://discuss.codemirror.net/t/codemirror-6-single-line-and-or-avoid-carriage-return/2979/2
            EditorState.transactionFilter.of(tr => tr.newDoc.lines > 1 ? [] : tr),
            EditorView.domEventHandlers({
                paste: (event, view) => {
                    const content = event.clipboardData.getData('text/plain')

                    if(content.includes('\n')) {
                        event.preventDefault()
                        const contentWithoutNewLines = content.replace(/[\n\r]/g, '')
                        const transaction = view.state.replaceSelection(contentWithoutNewLines)
                        const update = view.state.update(transaction)
                        view.update([update])
                    }
                }
            }),
            keymap.of([
                ...historyKeymap
            ]),
            placeholder(vueInstance.placeholder)
        ]
    })
}

export default {
    props: {
        modelValue: {
            type: String,
            default: ''
        },
        placeholder: {
            type: String,
            default: ''
        }
    },
    data() {
        return {
            editor: null,
            emitted: false
        }
    },
    watch: {
        modelValue() {
            if(!this.emitted) {
                this.editor.setState(createState(this))
            } else {
                this.emitted = false
            }
        }
    },
    mounted() {
        this.editor = new EditorView({
            state: createState(this),
            parent: this.$el
        })
    }
}
</script>

<style>
.code-mirror-single-line .cm-editor.cm-focused {
    outline: 0 !important;
}

.code-mirror-single-line .cm-scroller {
    font-family: inherit !important;
    margin-left: 0.2rem;
    margin-right: 0.5rem;
    overflow-x: hidden !important;
}
</style>
