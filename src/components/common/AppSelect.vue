<template>
  <div class="common-select" ref="selectRef">
    <div
      class="select-control"
      :class="{
        'is-open': isOpen,
        'is-disabled': disabled,
        'is-focused': isFocused,
        searchable: showSearch,
      }"
      tabindex="0"
      @mouseenter="isHover = true"
      @mouseleave="isHover = false"
      @click="handleControlClick"
      @keydown="handleControlKeydown"
    >
      <div class="select-content">
        <!-- Selected Values Display -->
        <div class="select-values" @click="handleValuesClick">
          <!-- Multiple Mode: Tags -->
          <template v-if="multiple && selectedValues.length > 0">
            <span v-for="val in selectedValues" :key="val" class="tag">
              {{ getOptionLabel(val) }}
              <span class="tag-close" @click.stop="removeValue(val)">×</span>
            </span>
          </template>

          <!-- Single Mode: Label (only when not searching) -->
          <template v-if="showSingleLabel">
            <span class="single-value">{{ getOptionLabel(selectedValues[0]) }}</span>
          </template>

          <!-- Input (search) -->
          <input
            v-if="showSearch"
            ref="searchInputRef"
            type="text"
            class="search-input"
            v-model="searchKeyword"
            :placeholder="inputPlaceholder"
            :disabled="disabled"
            @input="handleSearchInput"
            @focus="handleSearchFocus"
            @blur="handleSearchBlur"
            @click.stop
          />

          <!-- Placeholder -->
          <span v-if="!hasSelection && !showSearch" class="placeholder">
            {{ placeholder }}
          </span>
        </div>
      </div>

      <span v-if="activeIcon === 'search'" class="icon">
        <svg
          viewBox="64 64 896 896"
          focusable="false"
          data-icon="search"
          width="1em"
          height="1em"
          fill="currentColor"
          aria-hidden="true"
        >
          <path
            d="M909.6 854.5L649.9 594.8C690.2 542.7 712 479 712 412c0-80.2-31.3-155.4-87.9-212.1-56.6-56.7-132-87.9-212.1-87.9s-155.5 31.3-212.1 87.9C143.2 256.5 112 331.8 112 412c0 80.1 31.3 155.5 87.9 212.1C256.5 680.8 331.8 712 412 712c67 0 130.6-21.8 182.7-62l259.7 259.6a8.2 8.2 0 0011.6 0l43.6-43.5a8.2 8.2 0 000-11.6zM570.4 570.4C528 612.7 471.8 636 412 636s-116-23.3-158.4-65.6C211.3 528 188 471.8 188 412s23.3-116.1 65.6-158.4C296 211.3 352.2 188 412 188s116.1 23.2 158.4 65.6S636 352.2 636 412s-23.3 116.1-65.6 158.4z"
          ></path>
        </svg>
      </span>

      <span v-if="activeIcon === 'arrow'" class="icon icon-arrow" :class="{ 'is-open': isOpen }">
        <svg width="1em" height="1em" viewBox="0 0 12 12" fill="none">
          <path
            d="M3 4.5L6 7.5L9 4.5"
            stroke="currentColor"
            stroke-width="1.5"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
        </svg>
      </span>

      <span v-if="activeIcon === 'clear'" class="icon icon-clear" @click.stop="clearSelection">
        <svg
          fill-rule="evenodd"
          viewBox="64 64 896 896"
          focusable="false"
          data-icon="close-circle"
          width="1em"
          height="1em"
          fill="currentColor"
          aria-hidden="true"
        >
          <path
            d="M512 64c247.4 0 448 200.6 448 448S759.4 960 512 960 64 759.4 64 512 264.6 64 512 64zm127.98 274.82h-.04l-.08.06L512 466.75 384.14 338.88c-.04-.05-.06-.06-.08-.06a.12.12 0 00-.07 0c-.03 0-.05.01-.09.05l-45.02 45.02a.2.2 0 00-.05.09.12.12 0 000 .07v.02a.27.27 0 00.06.06L466.75 512 338.88 639.86c-.05.04-.06.06-.06.08a.12.12 0 000 .07c0 .03.01.05.05.09l45.02 45.02a.2.2 0 00.09.05.12.12 0 00.07 0c.02 0 .04-.01.08-.05L512 557.25l127.86 127.87c.04.04.06.05.08.05a.12.12 0 00.07 0c.03 0 .05-.01.09-.05l45.02-45.02a.2.2 0 00.05-.09.12.12 0 000-.07v-.02a.27.27 0 00-.05-.06L557.25 512l127.87-127.86c.04-.04.05-.06.05-.08a.12.12 0 000-.07c0-.03-.01-.05-.05-.09l-45.02-45.02a.2.2 0 00-.09-.05.12.12 0 00-.07 0z"
          ></path>
        </svg>
      </span>
    </div>

    <!-- Dropdown -->
    <teleport to="body">
      <transition name="dropdown">
        <div v-if="isOpen" ref="dropdownRef" class="select-dropdown" :style="dropdownStyle">
          <!-- Select All (only for multiple mode) -->
          <div
            v-if="multiple && showSelectAll"
            class="select-option select-all"
            @click="handleSelectAllClick"
          >
            <input type="checkbox" :checked="isAllSelected" @click.stop="handleSelectAllClick" />
            <span>Select All</span>
          </div>

          <!-- Options List -->
          <div class="options-list">
            <div
              v-for="(option, index) in filteredOptions"
              :key="option.value"
              class="select-option"
              :class="{
                'is-selected': isSelected(option.value),
                'is-disabled': option.disabled,
                'is-focused': focusedIndex === index,
              }"
              @click="handleOptionClick(option)"
              @mouseenter="handleOptionMouseEnter(index)"
            >
              <input
                v-if="multiple"
                type="checkbox"
                :checked="isSelected(option.value)"
                :disabled="option.disabled"
              />
              <span class="option-label">{{ option.label }}</span>
            </div>

            <div v-if="filteredOptions.length === 0" class="no-results">No results found</div>
          </div>
        </div>
      </transition>
    </teleport>
  </div>
</template>

<script setup lang="ts">
  import { ref, computed, watch, onMounted, onUnmounted, nextTick } from 'vue'

  interface Option {
    label: string
    value: string | number
    disabled?: boolean
  }

  interface Props {
    modelValue?: string | number | Array<string | number> | null
    options: Option[]
    placeholder?: string
    disabled?: boolean
    allowClear?: boolean
    showSearch?: boolean
    multiple?: boolean
    showSelectAll?: boolean
  }

  const props = withDefaults(defineProps<Props>(), {
    modelValue: null,
    placeholder: 'Select',
    disabled: false,
    allowClear: false,
    showSearch: false,
    multiple: false,
    showSelectAll: false,
  })

  const emit = defineEmits<{
    'update:modelValue': [value: string | number | Array<string | number> | null]
    change: [value: string | number | Array<string | number> | null]
    search: [keyword: string]
    open: []
    close: []
  }>()

  // Refs
  const selectRef = ref<HTMLDivElement | null>(null)
  const searchInputRef = ref<HTMLInputElement | null>(null)
  const isOpen = ref(false)
  const isInputFocused = ref(false)
  const isHover = ref(false)
  const searchKeyword = ref('')
  const focusedIndex = ref(0)
  const dropdownRef = ref<HTMLDivElement | null>(null)
  const dropdownStyle = ref<Record<string, string>>({})

  const isFocused = computed(() => {
    return isOpen.value || isInputFocused.value
  })

  // Normalize modelValue to array internally
  const selectedValues = computed<Array<string | number>>(() => {
    if (props.modelValue === null || props.modelValue === undefined) {
      return []
    }
    return Array.isArray(props.modelValue) ? props.modelValue : [props.modelValue]
  })

  const hasSelection = computed<boolean>(() => selectedValues.value.length > 0)

  const canClear = computed<boolean>(() => {
    if (!props.allowClear || props.disabled) return false
    if (props.multiple) return props.modelValue !== null
    return selectedValues.value.length > 0
  })

  const activeIcon = computed(() => {
    if (canClear.value && isHover.value) return 'clear'
    if (props.showSearch) return 'search'
    return 'arrow'
  })

  // Filter options based on search keyword
  const filteredOptions = computed<Option[]>(() => {
    if (!searchKeyword.value.trim()) {
      return props.options
    }
    const keyword = searchKeyword.value.toLowerCase()
    return props.options.filter((option) => {
      const labelMatch = option.label.toLowerCase().includes(keyword)
      const valueMatch = String(option.value).toLowerCase().includes(keyword)
      return labelMatch || valueMatch
    })
  })

  const showSingleLabel = computed<boolean>(() => {
    if (props.multiple) return false
    if (selectedValues.value.length === 0) return false

    if (props.showSearch) {
      return !isInputFocused.value && searchKeyword.value === ''
    }

    return true
  })

  const inputPlaceholder = computed<string>(() => {
    if (!hasSelection.value) {
      return props.placeholder
    }

    if (!props.multiple && props.showSearch && isInputFocused.value && searchKeyword.value === '') {
      return getOptionLabel(selectedValues.value[0])
    }

    return ''
  })

  // Check if all non-disabled options are selected
  const isAllSelected = computed(() => {
    const enabledOptions = props.options.filter((opt) => !opt.disabled)
    if (enabledOptions.length === 0) return false
    return enabledOptions.every((opt) => selectedValues.value.includes(opt.value))
  })

  // Helper functions
  const isSelected = (value: string | number): boolean => {
    return selectedValues.value.includes(value)
  }

  const getOptionLabel = (value: string | number): string => {
    const option = props.options.find((opt) => opt.value === value)
    return option ? option.label : String(value)
  }

  // Emit normalized value
  const emitValue = (values: Array<string | number>) => {
    let emitVal: string | number | Array<string | number> | null

    if (props.multiple) {
      emitVal = values
    } else {
      emitVal = values.length > 0 ? values[0] : null
    }

    emit('update:modelValue', emitVal)
    emit('change', emitVal)
  }

  const updateDropdownPosition = () => {
    if (!selectRef.value || !dropdownRef.value) return

    const controlRect = selectRef.value.getBoundingClientRect()
    const dropdownRect = dropdownRef.value.getBoundingClientRect()

    const viewportHeight = window.innerHeight
    const spaceBelow = viewportHeight - controlRect.bottom
    const spaceAbove = controlRect.top

    const shouldFlip = spaceBelow < dropdownRect.height && spaceAbove > spaceBelow

    dropdownStyle.value = {
      position: 'absolute',
      width: `${controlRect.width}px`,
      left: `${controlRect.left}px`,
      top: shouldFlip
        ? `${controlRect.top - dropdownRect.height - 4}px`
        : `${controlRect.bottom + 4}px`,
      zIndex: '9999',
    }
  }

  // Control handlers
  const handleControlClick = () => {
    if (props.disabled) return
    toggleDropdown()
  }

  const handleValuesClick = (e: PointerEvent) => {
    if (props.showSearch && searchInputRef.value) {
      e.stopPropagation()
      searchInputRef.value.focus()
    }
  }

  // Search handlers
  const handleSearchInput = () => {
    emit('search', searchKeyword.value)
    focusedIndex.value = 0
    if (!isOpen.value) {
      openDropdown()
    }
  }

  const handleSearchFocus = () => {
    isInputFocused.value = true
    isOpen.value = true
    initFocusedIndex()
  }

  const handleSearchBlur = () => {
    isInputFocused.value = false
  }

  const clearSearchKeyword = () => {
    searchKeyword.value = ''
  }

  const initFocusedIndex = () => {
    if (hasSelection.value) {
      const firstSelectedValueIndex = props.options.findIndex(
        (option) => !option.disabled && selectedValues.value.includes(option.value)
      )

      focusedIndex.value = firstSelectedValueIndex
    } else {
      focusedIndex.value = 0
    }
  }

  // Dropdown control
  const openDropdown = () => {
    if (props.disabled || isOpen.value) return
    isOpen.value = true
    initFocusedIndex()
    emit('open')
  }

  const closeDropdown = () => {
    if (!isOpen.value) return
    isOpen.value = false
    focusedIndex.value = 0
    emit('close')
  }

  const toggleDropdown = () => {
    if (isOpen.value) {
      closeDropdown()
    } else {
      openDropdown()
    }
  }

  const commitSelection = () => {
    // Thoát search mode hoàn toàn
    searchKeyword.value = ''
    isInputFocused.value = false

    // Blur input để tránh browser auto focus
    searchInputRef.value?.blur()
  }

  // Option selection
  const handleOptionClick = (option: Option) => {
    if (option.disabled) return

    let newValues: Array<string | number>

    if (props.multiple) {
      if (isSelected(option.value)) {
        newValues = selectedValues.value.filter((v) => v !== option.value)
      } else {
        newValues = [...selectedValues.value, option.value]
      }
    } else {
      newValues = [option.value]
      closeDropdown()
    }

    emitValue(newValues)
    clearSearchKeyword()
    commitSelection()
  }

  const removeValue = (value: string | number) => {
    const newValues = selectedValues.value.filter((v) => v !== value)
    emitValue(newValues)
  }

  const clearSelection = (e?: MouseEvent) => {
    e?.stopPropagation()

    if (!canClear.value) return

    const payload = props.multiple ? [] : null

    emit('update:modelValue', payload)
    emit('change', payload)

    searchKeyword.value = ''
    closeDropdown()
  }

  const handleSelectAllClick = () => {
    if (isAllSelected.value) {
      emitValue([])
    } else {
      const enabledOptions = props.options.filter((opt) => !opt.disabled)
      const allValues = enabledOptions.map((opt) => opt.value)
      emitValue(allValues)
    }

    clearSearchKeyword()
  }

  // Mouse hover
  const handleOptionMouseEnter = (index: number) => {
    focusedIndex.value = index
  }

  // Keyboard navigation
  const handleControlKeydown = (e: KeyboardEvent) => {
    if (props.disabled) return

    // ENTER chọn option trước
    if (isOpen.value && e.key === 'Enter' && focusedIndex.value >= 0) {
      e.preventDefault()
      const option = filteredOptions.value[focusedIndex.value]
      if (option && !option.disabled) {
        handleOptionClick(option)
      }
      return
    }

    // ENTER / SPACE mở dropdown
    if ((e.key === 'Enter' || e.key === ' ') && !isOpen.value) {
      e.preventDefault()
      openDropdown()
      return
    }

    if (e.key === 'Escape') {
      e.preventDefault()
      closeDropdown()
      commitSelection()
      return
    }

    if (isOpen.value && (e.key === 'ArrowDown' || e.key === 'ArrowUp')) {
      e.preventDefault()
      handleArrowNavigation(e.key)
    }
  }

  const handleArrowNavigation = (key: 'ArrowDown' | 'ArrowUp') => {
    const options = filteredOptions.value
    if (options.length === 0) return

    let newIndex = focusedIndex.value

    const maxIndex = options.length - 1

    if (key === 'ArrowDown') {
      do {
        newIndex = newIndex >= maxIndex ? 0 : newIndex + 1
      } while (options[newIndex]?.disabled && newIndex !== focusedIndex.value)
    } else if (key === 'ArrowUp') {
      do {
        newIndex = newIndex <= 0 ? maxIndex : newIndex - 1
      } while (options[newIndex]?.disabled && newIndex !== focusedIndex.value)
    }

    focusedIndex.value = newIndex
    scrollOptionIntoView(newIndex)
  }

  // Scroll option to view if needed
  const scrollOptionIntoView = (index: number) => {
    const optionsListEl = dropdownRef.value?.querySelector<HTMLElement>('.options-list')
    if (!optionsListEl) return
    const optionEl = optionsListEl.children[index] as HTMLElement
    if (!optionEl) return

    const optionTop = optionEl.offsetTop
    const optionBottom = optionTop + optionEl.offsetHeight
    const scrollTop = optionsListEl.scrollTop
    const containerHeight = optionsListEl.clientHeight

    if (optionTop < scrollTop) {
      optionsListEl.scrollTop = optionTop
    } else if (optionBottom > scrollTop + containerHeight) {
      optionsListEl.scrollTop = optionBottom - containerHeight
    }
  }

  const handleClickOutside = (e: MouseEvent) => {
    const target = e.target as Node

    if (selectRef.value?.contains(target) || dropdownRef.value?.contains(target)) {
      return
    }

    closeDropdown()
  }

  watch(
    () => searchKeyword.value,
    () => {
      focusedIndex.value = -1
    }
  )

  watch(
    () => filteredOptions.value,
    () => {
      focusedIndex.value = -1
    }
  )

  watch(isOpen, async (open) => {
    if (open) {
      await nextTick()
      updateDropdownPosition()

      window.addEventListener('resize', updateDropdownPosition)
      window.addEventListener('scroll', updateDropdownPosition, true)

      if (selectRef.value) {
        resizeObserver?.observe(selectRef.value)
      }

      if (dropdownRef.value) {
        resizeObserver?.observe(dropdownRef.value)
      }
    } else {
      window.removeEventListener('resize', updateDropdownPosition)
      window.removeEventListener('scroll', updateDropdownPosition, true)

      resizeObserver?.disconnect()
    }
  })

  let resizeObserver: ResizeObserver | null = null

  onMounted(() => {
    resizeObserver = new ResizeObserver(() => {
      updateDropdownPosition()
    })
  })

  onUnmounted(() => {
    resizeObserver?.disconnect()
  })

  onMounted(() => {
    document.addEventListener('mousedown', handleClickOutside)
  })

  onUnmounted(() => {
    document.removeEventListener('mousedown', handleClickOutside)
  })
</script>

<style scoped lang="scss">
  .common-select {
    position: relative;
    width: 100%;
    height: fit-content;
    font-size: 14px;
    color: #000000;
  }

  .select-control {
    position: relative;
    display: flex;
    align-items: center;
    min-height: 30px;
    padding: 4px 10px;
    background: #ffffff;
    border: 1px solid #d9d9d9;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.2s;
    gap: 6px;

    &.searchable {
      cursor: text;
    }

    &:hover:not(.is-disabled) {
      border-color: #787878;
    }

    &.is-focused:not(.is-disabled) {
      // legacy version used background: #fff8e5 highlight on focus
      // modern UX: focus is indicated by border / shadow only
      border-color: #1677ff;
      box-shadow: 0 0 0 2px rgba(22, 119, 255, 0.2);
      background: #fff;
      outline: none;
    }

    &.is-disabled {
      background: #f5f5f5;
      cursor: not-allowed;
      opacity: 0.6;
    }
  }

  .icon {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    color: #8c8c8c;
  }

  .icon-arrow {
    margin-left: auto;
    transition: transform 0.2s;

    &.is-open {
      transform: rotate(180deg);
    }
  }

  .icon-clear {
    transition: all 0.2s;
    color: #00000040;
    cursor: pointer;

    &:hover {
      color: #00000073;
    }

    &:active {
      color: #000000e0;
    }
  }

  .select-content {
    flex: 1;
    display: flex;
    align-items: center;
    min-width: 0;
    position: relative;
  }

  .select-values {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    gap: 4px;
    width: 100%;
    min-height: 22px;
    position: relative;
  }

  .single-value {
    position: absolute;
    left: 0;
    right: 0;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    pointer-events: none;

    // Ẩn khi input đang focus hoặc có search keyword
    .dls-select.is-focused &,
    .select-values:has(.search-input:focus) & {
      opacity: 0;
    }
  }

  .search-input {
    flex: 1;
    width: 100%;
    min-width: 0;
    border: none;
    outline: none;
    background: transparent;
    font-size: 14px;
    padding: 2px 0;
    position: relative; // Để input luôn ở trên single-value
    z-index: 1;

    &::placeholder {
      color: #bfbfbf;
    }
  }

  .single-value {
    max-width: 100%;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    pointer-events: none;
  }

  .placeholder {
    color: #bfbfbf;
  }

  .tag {
    display: inline-flex;
    align-items: center;
    gap: 4px;
    padding: 2px 6px;
    background: #f0f0f0;
    border: 1px solid #d9d9d9;
    border-radius: 3px;
    font-size: 13px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .tag-close {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 14px;
    height: 14px;
    cursor: pointer;
    border-radius: 2px;
    font-size: 16px;
    line-height: 1;
    color: #787878;
    flex-shrink: 0;

    &:hover {
      background: #d9d9d9;
      color: #000000;
    }
  }

  .select-dropdown {
    background: #ffffff;
    border: 1px solid #d9d9d9;
    border-radius: 4px;
    box-shadow: 0 2px 8px #0000001a;
    max-height: 250px;
    overflow: hidden;
    display: flex;
    flex-direction: column;
  }

  .select-all {
    border-bottom: 1px solid #f0f0f0;
    font-weight: 500;
  }

  .options-list {
    display: flex;
    flex-direction: column;
    padding: 4px;
    overflow-y: auto;
    flex: 1;
    scrollbar-width: thin;
    scrollbar-color: rgba(0, 0, 0, 0.2) transparent;

    &::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }

    &::-webkit-scrollbar-track {
      background: transparent;
      border-radius: 3px;
    }

    &::-webkit-scrollbar-thumb {
      background-color: rgba(0, 0, 0, 0.2);
      border-radius: 3px;

      &:hover {
        background-color: rgba(0, 0, 0, 0.35);
      }
    }

    &::-webkit-scrollbar-button {
      display: none;
    }
  }

  .select-option {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 5px 12px;
    border-radius: 4px;
    cursor: pointer;
    transition: background 0.15s;

    &:hover:not(.is-disabled):not(.is-selected) {
      background: rgba(0, 0, 0, 0.04);
    }

    &.is-selected {
      background: #e6f4ff;
      font-weight: 600;

      &:hover:not(.is-disabled) {
        background: #e6f4ff;
      }
    }

    &.is-focused:not(.is-disabled):not(.is-selected) {
      background: rgba(0, 0, 0, 0.04);
    }

    &.is-disabled {
      color: #bfbfbf;
      cursor: not-allowed;
    }
  }

  .option-label {
    flex: 1;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .no-results {
    padding: 12px;
    text-align: center;
    color: #8c8c8c;
  }

  input[type='checkbox'] {
    width: 16px;
    height: 16px;
    cursor: pointer;
    flex-shrink: 0;

    .is-disabled & {
      cursor: not-allowed;
    }
  }

  .dropdown-enter-active,
  .dropdown-leave-active {
    transition:
      opacity 0.2s,
      transform 0.2s;
  }

  .dropdown-enter-from,
  .dropdown-leave-to {
    opacity: 0;
    transform: translateY(-4px);
  }
</style>
