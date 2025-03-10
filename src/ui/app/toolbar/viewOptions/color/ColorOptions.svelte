<script lang="ts">
  import produce from "immer";
  import { dndzone } from "svelte-dnd-action";
  import {
    Button,
    IconButton,
    Icon,
    Select,
    TextInput,
    NumberInput,
    ColorInput,
    Checkbox,
    // DateInput, //use native date input temporarily,
  } from "obsidian-svelte";
  import { TagsInput } from "src/ui/components/TagsInput";
  import HorizontalGroup from "src/ui/components/HorizontalGroup/HorizontalGroup.svelte";
  import type { DataField } from "src/lib/dataframe/dataframe";
  import {
    filterOperatorTypes,
    isNumberFilterOperator,
    isStringFilterOperator,
    isDateFilterOperator,
    isListFilterOperator,
    type ColorFilterDefinition,
    type FilterOperator,
  } from "src/settings/settings";
  import {
    setColor,
    type ColorRuleWithId,
    setField,
    setOperator,
    withIds,
    setValue,
    setEnabled,
    removeCondition,
    addCondition,
    getFieldByName,
    getOperatorsByField,
    stripIds,
  } from "./helpers";
  import { fieldsToSelectOptions } from "../helpers";
  import { i18n } from "src/lib/stores/i18n";

  export let filter: ColorFilterDefinition;
  export let fields: DataField[];
  export let onFilterChange: (filter: ColorFilterDefinition) => void;

  $: fieldOptions = fieldsToSelectOptions(fields);

  $: rules = withIds(filter.conditions);

  const handleColorChange = (i: number) => (event: Event) => {
    if (event.currentTarget instanceof HTMLInputElement) {
      filter = setColor(filter, i, event.currentTarget.value);
      onFilterChange(filter);
    }
  };

  const handleFieldChange =
    (i: number) =>
    ({ detail }: CustomEvent<string>) => {
      filter = setField(filter, i, detail);
      onFilterChange(filter);
    };

  const handleOperatorChange =
    (i: number) =>
    ({ detail }: CustomEvent<string>) => {
      filter = setOperator(filter, i, detail as FilterOperator);
      onFilterChange(filter);
    };

  const handleValueChange = (i: number) => (event: FocusEvent) => {
    if (event.currentTarget instanceof HTMLInputElement) {
      filter = setValue(filter, i, event.currentTarget.value);
      onFilterChange(filter);
    }
  };

  const handleStatusChange =
    (i: number) =>
    ({ detail }: CustomEvent<boolean>) => {
      filter = setEnabled(filter, i, detail);
      onFilterChange(filter);
    };

  const handleConditionRemove = (i: number) => (event: MouseEvent) => {
    event.stopPropagation();
    filter = removeCondition(filter, i);
    onFilterChange(filter);
  };

  function handleConditionAdd() {
    filter = addCondition(filter, fields);
    onFilterChange(filter);
  }

  const flipDurationMs = 200;

  function handleDndConsider(e: CustomEvent<DndEvent<ColorRuleWithId>>) {
    rules = e.detail.items;
  }

  function handleDndFinalize(e: CustomEvent<DndEvent<ColorRuleWithId>>) {
    filter = produce(filter, (draft) => {
      draft.conditions = stripIds(e.detail.items);
    });
    onFilterChange(filter);
  }
</script>

<div style="display: flex; flex-direction: column; gap: 8px;">
  <div
    style="display: flex; flex-direction: column; gap: 8px;"
    use:dndzone={{
      type: "color conditions",
      items: rules,
      flipDurationMs,
      dropTargetStyle: {
        outline: "none",
        borderRadius: "5px",
        background: "hsla(var(--interactive-accent-hsl), 0.3)",
        transition: "all 150ms easy-in-out",
      },
    }}
    on:consider={handleDndConsider}
    on:finalize={handleDndFinalize}
  >
    {#each rules as rule, i (rule.id)}
      {@const field = getFieldByName(fields, rule.condition.field)}
      <HorizontalGroup>
        <Icon name="grip-vertical" />
        <ColorInput value={rule.color} on:change={handleColorChange(i)} />
        <div class="setting-item-name">
          {$i18n.t("components.color.where")}
        </div>
        <Select
          value={rule.condition.field}
          options={fieldOptions}
          on:change={handleFieldChange(i)}
        />
        <Select
          value={rule.condition.operator}
          on:change={handleOperatorChange(i)}
          options={field ? getOperatorsByField(field) : []}
        />

        {#if filterOperatorTypes[rule.condition.operator] === "binary"}
          {#if isStringFilterOperator(rule.condition.operator)}
            <TextInput
              value={rule.condition.value ?? ""}
              on:blur={handleValueChange(i)}
            />
          {:else if isNumberFilterOperator(rule.condition.operator)}
            <NumberInput
              value={parseFloat(rule.condition.value ?? "")}
              on:blur={handleValueChange(i)}
            />
          {:else if isDateFilterOperator(rule.condition.operator)}
            <input
              type="date"
              value={rule.condition.value ?? ""}
              on:blur={handleValueChange(i)}
              max="9999-12-31"
            />
          {:else if isListFilterOperator(rule.condition.operator)}
            <TagsInput
              strict={rule.condition.field === "tags"}
              unique={true}
              value={JSON.parse(rule.condition.value ?? "[]")}
              on:change={(event) => {
                filter = setValue(filter, i, event.detail);
                onFilterChange(filter);
              }}
            />
          {/if}
        {/if}
        <Checkbox
          checked={rule.condition?.enabled ?? true}
          on:check={handleStatusChange(i)}
        />
        <IconButton icon="trash" onClick={handleConditionRemove(i)} />
      </HorizontalGroup>
    {/each}
  </div>
  <HorizontalGroup>
    <Button variant="plain" on:click={handleConditionAdd}
      ><Icon name="plus" />{$i18n.t("components.color.add")}</Button
    >
  </HorizontalGroup>
</div>
