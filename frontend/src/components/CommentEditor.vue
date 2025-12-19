<script>
import { EditorContent } from '@tiptap/vue-3'
import TextEditor from '@/components/TextEditor.vue'
import { TextEditorFixedMenu } from 'frappe-ui/src/components/TextEditor'

export default {
  name: 'CommentEditor',

  components: { TextEditor, TextEditorFixedMenu, EditorContent },

  props: {
    value: { type: String, default: '' },
    placeholder: { type: String, default: null },
    editable: { type: Boolean, default: true },
    editorProps: { type: Object, default: () => ({}) },
    submitButtonProps: { type: Object, default: () => ({}) },
    discardButtonProps: { type: Object, default: () => ({}) },
  },

  emits: ['change', 'rich-quote', 'rich-quote-click'],
  expose: ['editor'],

  computed: {
    editor() {
      return this.$refs.textEditor?.editor
    },

    textEditorMenuButtons() {
      return [
        'Paragraph',
        ['Heading 2', 'Heading 3', 'Heading 4', 'Heading 5', 'Heading 6'],
        'Separator',
        'Bold',
        'Italic',
        'Separator',
        'Bullet List',
        'Numbered List',
        'Separator',
        'Align Left',
        'Align Center',
        'Align Right',
        'FontColor',
        'Separator',
        'Image',
        'Video',
        'Attachment',
        'Iframe',
        'Link',
        'Blockquote',
        'Code',
        'Horizontal Rule',
        [
          'InsertTable',
          'AddColumnBefore',
          'AddColumnAfter',
          'DeleteColumn',
          'AddRowBefore',
          'AddRowAfter',
          'DeleteRow',
          'MergeCells',
          'SplitCell',
          'ToggleHeaderColumn',
          'ToggleHeaderRow',
          'ToggleHeaderCell',
          'DeleteTable',
        ],
      ]
    },
  },

  mounted() {
    this.registerAttachmentCommand()
  },

  methods: {
    registerAttachmentCommand() {
      const editor = this.editor
      if (!editor) return

      editor.commands.attachment = () => {
        const input = document.createElement('input')
        input.type = 'file'
        input.accept = '.pdf,.xls,.xlsx'

        input.onchange = async () => {
          const file = input.files[0]
          if (!file) return

          const formData = new FormData()
          formData.append('file', file)
          formData.append('is_private', 0)

          const res = await fetch('/api/method/upload_file', {
            method: 'POST',
            body: formData,
            credentials: 'include',
          })

          const data = await res.json()
          const fileUrl = data.message.file_url

          editor
            .chain()
            .focus()
            .insertContent(
              `<p><a href="${fileUrl}" target="_blank">📎 ${file.name}</a></p>`
            )
            .run()
        }

        input.click()
        return true
      }
    },
  },
}
</script>
