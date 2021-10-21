```tsx
import { computed, defineComponent, reactive, ref, toRef, PropType, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';

import { feedbackAdd } from '@/api/monitor';

import { to } from '~/util'
import { useStore } from "vuex";
import uploadImg from '~/components/upload-img'

interface FormState {
  content: string | undefined;
  files: string[];
  title?: string
}

export default defineComponent({
  name: 'feedBackAddModal',
  components: {
    uploadImg,
  },
  props: {
    visible: { type: Boolean as PropType<boolean>, default: false },
  },
  emits: {
    'update:visible': (show: boolean) => show,
  },
  setup(props, { emit }) {
    const route = useRoute();
    const router = useRouter();
    const store = useStore();

    const id = computed<string>(() => route.params.id as string);

    const userName = toRef(store.state, 'userName');

    let show = ref<boolean>(false);
    let confirmLoading = ref<boolean>(false);

    watch(show, () => emit('update:visible', show.value));
    watch(() => props.visible, (val) => show.value = val);

    const formData = reactive<FormState>({
      content: '',
      files: [],
    });

    const handleOk = async () => {
      confirmLoading.value = true
      const [err, res] = await to(
        feedbackAdd(formData)
      )
      if (err) {
        console.log(err)
        return
      }
      if (res.code === 0) {
        show.value = false
        confirmLoading.value = false
        return router.push({ name: 'feedBackDetail', params: { id: id.value } });
      } else {
      }
    };

    return {
      formData,
      show,
      confirmLoading,
      userName,
      handleOk,
    };
  },
  render() {
    return (
      <a-modal
        title={'投诉建议'}
        v-model={this.show}
        confirmLoading={this.confirmLoading}
        onOk={this.handleOk}
      >
        <a-form-model
          model={this.formData}
          ref="ruleForm"
          label-col={{ span: 5 }}
          wrapper-col={{ span: 16 }}
        >
          <a-form-model-item label="姓名">
            { this.userName }
          </a-form-model-item>
          <a-form-model-item label="描述" prop="content">
            <a-textarea
              v-model={this.formData.content}
              placeholder="请在此输入您想投诉的问题，我们会第一时间为您处理"
              auto-size={{ minRows: 4 }} />
          </a-form-model-item>
          <a-form-model-item label="图片上传">
            <uploadImg multiple number={9} onOchange={(files: string[]) => this.formData.files = files}/>
          </a-form-model-item>
        </a-form-model>
      </a-modal>
    )}
})


```

