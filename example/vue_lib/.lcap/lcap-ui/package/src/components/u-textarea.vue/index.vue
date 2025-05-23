<template>
    <div
        v-if="!isPreview"
        :class="$style.root"
        :readonly="readonly"
        :disabled="disabled"
        :style="{ width, height }"
        :color="currentColor || (formItemVM && formItemVM.color)"
        :focus="focused"
        @click.self="!focused && focus()"
    >
        <span :class="$style.baseline">b</span>
        <!-- 用于基线对齐 -->
        <span :class="$style.placeholder" v-if="placeholder">{{ valueEmpty ? placeholder : '' }}</span>
        <textarea
            ref="input"
            :class="$style.input"
            v-bind="$attrs"
            :value="currentValue"
            v-focus="autofocus"
            :readonly="readonly"
            :disabled="disabled"
            @input="onInput"
            @focus="onFocus"
            @blur="onBlur"
            @keypress="onKeypress"
            v-on="listeners"
            @compositionstart="compositionInputing = true"
            @compositionend="onCompositionEnd"
            :maxlength="maxlength"
        ></textarea>
        <slot></slot>
        <span v-if="clearable && !valueEmpty" :class="$style.clearable" @click.stop="clear"></span>
        <f-dragger @dragstart="onDragStart" @drag="onDrag">
            <div ref="handle" :class="$style.handle" v-show="resize !== 'none'" :resize="resize"></div>
        </f-dragger>
        <span v-if="showWordLimit && maxlength" :class="limitPosition === 'inside' ? $style.limit : $style.limitOut">{{ limit }}/{{ maxlength }}</span>
    </div>
    <u-preview v-else :text="currentValue"></u-preview>
</template>

<script>
import UInput from '../u-input.vue';
import { getSize } from '../../utils/dom';

export default {
    name: 'u-textarea',
    extends: UInput,
    props: {
        showWordLimit: Boolean,
        limitPosition: {
            type: String,
            default: 'inside',
            validator: (value) => ['inside', 'outside'].includes(value),
        },
        maxlength: [String, Number],
        autosize: [Boolean, Object],
        resize: {
            type: String,
            default: 'vertical',
            validator: (value) => ['none', 'horizontal', 'vertical', 'both'].includes(value),
        },
        initHeight: { type: String, default: '' },

    },
    data() {
        return { startWidth: 0, startHeight: 0, width: '', height: this.initHeight, canAuto: true };
    },
    computed: {
        limit() {
            return String(this.currentValue || '').length;
        },
    },
    watch: {
        currentValue() {
            this.$nextTick(this.adjustSize);
        },
    },
    methods: {
        onDragStart() {
            const size = getSize(this.$el);
            this.startWidth = size.width;
            this.startHeight = size.height;
            this.canAuto = false;
            const { input } = this.$refs;
            input.style.height = null;
        },
        onDrag($event) {
            if (this.resize === 'horizontal' || this.resize === 'both') {
                this.width = this.startWidth + $event.dragX + 'px';
            }
            if (this.resize === 'vertical' || this.resize === 'both') {
                this.height = this.startHeight + $event.dragY + 'px';
            }
            this.$refs.handle.style.left = 'auto';
            this.$refs.handle.style.top = 'auto';
        },
        autoResize() {
            const inputEl = this.$refs.input;
            if (this.autoSize === 'both' || this.autoSize === 'horizontal') {
                inputEl.style.width = '3px';
                this.width = inputEl.scrollWidth + (this.$el.offsetWidth - this.$el.clientWidth) + 'px';
                inputEl.style.width = '';
            }
            if (this.autoSize === 'both' || this.autoSize === 'vertical') {
                inputEl.style.height = '3px';
                this.height = inputEl.scrollHeight + (this.$el.offsetHeight - this.$el.clientHeight) + 'px';
                inputEl.style.height = '';
            }
        },
        isObject(val) {
            return val !== null && typeof val === 'object';
        },
        adjustSize() {
            if (!this.canAuto) {
                return;
            }
            const { input } = this.$refs;
            let height = input.scrollHeight;
            if (this.autosize === true || this.isObject(this.autosize)) {
                if (this.isObject(this.autosize || input.autosize)) {
                    const { maxHeight, minHeight } = this.autosize || input.autosize;
                    input.style.height = 'auto';
                    if (maxHeight) {
                        height = Math.min(height, maxHeight);
                    }
                    if (minHeight) {
                        height = Math.max(height, minHeight);
                    }
                    input.style.overflowY = 'auto';
                }
                if (height) {
                    const tempH = height + 'px';
                    input.style.height = tempH;
                    this.height = height + 2 * 1 + 'px';
                }
            }
        },
        handleEmptyValue(value) {
            if (!this.emptyValueIsNull) {
                return value;
            } else {
              return value ? value : null;
           }
        }
    },
};
</script>

<style module>
@import '../u-input.vue/index.css';

.root {
    width: var(--textarea-width);
    height: var(--textarea-height);
    min-width: var(--textarea-min-width);
    min-height: var(--textarea-min-height);
    padding: var(--textarea-padding);
    line-height: inherit;
    max-width: 100%;
    color: var(--textarea-color);
    border-color: var(--textarea-border-color);
    border-radius: var(--textarea-border-radius);
    transition: border-color var(--transition-duration-base);
}

.input {
    resize: none;
}

.clearable {
    position: absolute;
    right: 2px;
}

.clearable::before {
    top: 2px;
    right: 4px;
}

.handle {
    position: absolute;
    right: 1px;
    bottom: 1px;
    display: block;
    width: 8px;
    height: 8px;
    overflow: hidden;
    cursor: default;
}

.handle::before {
    content: '';
    display: block;
    height: 4px;
    width: 12px;
    border-top: 1px solid #666;
    border-bottom: 1px solid #666;
    transform: rotate(-45deg) translate(-2px, 2px);
}
.handle[resize='vertical'] {
    cursor: ns-resize;
}
.handle[resize='horizontal'] {
    cursor: ew-resize;
}
.handle[resize='both'] {
    cursor: nwse-resize;
}

.root[focus] {
    border-color: var(--textarea-border-color-focus);
    box-shadow: var(--textarea-box-shadow-focus);
}

.root[disabled] {
    background: var(--textarea-background-disabled);
    border-color: var(--textarea-border-color-disabled);
    color: var(--textarea-color-disabled);
}

.root[color='error'] {
    border-color: var(--textarea-border-color-error);
}

.root[size$='normal'] {
    width: var(--textarea-width);
}
.root[size^='normal'] {
    height: var(--textarea-height);
    line-height: inherit;
}

.root[size$='medium'] {
    width: var(--textarea-width-medium);
}
.root[size^='medium'] {
    height: var(--textarea-height-medium);
    line-height: inherit;
}

.root[size$='large'] {
    width: var(--textarea-width-large);
}
.root[size^='large'] {
    height: var(--textarea-height-large);
    line-height: inherit;
}

.root[size$='huge'] {
    width: var(--textarea-width-huge);
}
.root[size^='huge'] {
    height: var(--textarea-height-huge);
    line-height: inherit;
}

.root[size$='full'] {
    width: 100%;
    min-width: 0;
}
.root[size^='full'] {
    height: 100%;
}

.limit {
    bottom: 0;
    right: 10px;
    position: absolute;
    line-height: 1;
}

.limitOut {
    bottom: -16px;
    right: 0px;
    position: absolute;
    line-height: 1;
}
</style>
