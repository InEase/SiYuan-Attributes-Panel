<template>
    <div class="add-attribute-row">
        <t-button style="width: 150px" theme="default" variant="text" v-if="!creating" @click="creating = true">
            <template #icon><add-icon /></template>
            添加新的属性
        </t-button>

        <template v-else>
            <t-select v-model="attributeKey" :borderless="true" class="attr-selector" placeholder="-请选择-" :showArrow="false"
                filterable @change="handleSelect" creatable @create="handleCreate">
                <!-- <t-option v-for="item in options" :key="item.key" :value="item.key" :label="item.displayAs">
                    <div class="create-option">
                        <component :is="item.icon"></component>
                        <div>{{ item.displayAs }}</div>
                    </div>
                </t-option> -->

                <template #empty>
                    <div class="t-select__empty">没有更多预设啦<br> 输入以创建新值！</div>
                </template>

                <template #prefixIcon>
                    <component :is="dynamicIcon"></component>
                </template>
            </t-select>
            <!-- This is rendered by displayElement. -->
            <component :is="dynamicComponent"></component>
        </template>
    </div>
</template>
  
<script setup lang="tsx">
import { computed, ref, shallowRef } from 'vue';
import { useAttributesStore } from '@/store/attribute';
import { useConfigStore } from '@/store/rules';
import { AddIcon, ChevronDownIcon, ViewListIcon } from 'tdesign-icons-vue-next';
import { MessagePlugin } from 'tdesign-vue-next';

const attributeStore = useAttributesStore();
const configStore = useConfigStore();

// const avaliable = configStore.createOptions
// const existingRows = computed(() => attributeStore.builtInAttributes.map(item => item.key)
// const options = computed(() => {
//     return avaliable.filter(item => !existingRows.value.includes(item.key))
// })

const attributeKey = ref("");
const attributeValue = ref('');

const dynamicIcon = shallowRef(
    <ChevronDownIcon />
)

function submit(text, callback?: any) {
    attributeStore.setAttribute(attributeKey.value, text, () => {
        // TODO: 这里没做错误处理，万一没成功呢？
        if (callback && typeof callback === "function") callback()
        MessagePlugin.success(`添加成功`)
        // reset all states
        attributeKey.value = "";
        attributeValue.value = "";
        dynamicIcon.value = <ChevronDownIcon />
        dynamicComponent.value = <t-input
            defaultValue={attributeValue.value}
            autowidth
            borderless="true"
        />
        creating.value = false
    })
}

function handleSelect() {
    const displayRule = configStore.displayRules[attributeKey.value];
    if (displayRule) {
        const displayMethod = configStore.renderMethods[displayRule.dataType];
        dynamicIcon.value = displayRule.icon;
        if (displayMethod) {
            dynamicComponent.value = displayMethod("", true, submit);
        }
    } else {
        console.log("### ??? 怎么没有")
    }
}

function handleCreate(key: string) {
    // 需要以custom-开头
    if (!key.startsWith("custom-")) {
        MessagePlugin.error("自定义属性必须以custom-开头")
        return
    }
    // 目前只支持创建英文字段，中文字段不支持，后面跟类型一起支持？还是说，，直接转成拼音算了，不然还要多一步太麻烦了
    attributeKey.value = key
    dynamicIcon.value = <ViewListIcon />;
    dynamicComponent.value = <t-input
        autowidth
        borderless="true"
        onEnter={submit}
    />
}

const dynamicComponent = shallowRef(
    <t-input
        defaultValue={attributeValue.value}
        autowidth
        borderless="true"
    />
)

const creating = ref(false)
</script>

<style scoped>
.create-option {
    display: flex;
    align-items: center;
}

/* make select and input in one row */
.add-attribute-row {
    display: flex;
    align-items: center;
    margin-bottom: 8px;
}

:deep(.t-input:not(:hover)) {
    border: white;
}

:deep(.t-input--focused) {
    border-color: var(--td-brand-color);
}

.attr-selector {
    width: 150px;
    margin-right: 8px;
}

:deep(.t-icon) {
    margin-right: 4px;
}

:deep(.t-button--variant-text) {
    padding-left: 0px;
}
</style>