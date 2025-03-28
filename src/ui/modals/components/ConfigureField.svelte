<script lang="ts">
  import {
    Button,
    ModalButtonGroup,
    ModalContent,
    ModalLayout,
    Select,
    SettingItem,
    Switch,
    TextInput,
  } from "obsidian-svelte";
  import MultiTextInput from "src/ui/components/MultiTextInput/MultiTextInput.svelte";
  import { DataFieldType, type DataField } from "src/lib/dataframe/dataframe";
  import { i18n } from "src/lib/stores/i18n";

  export let title: string;
  export let field: DataField;
  export let editable: boolean;
  export let onSave: (field: DataField) => void;

  function handleNameChange(value: CustomEvent<string>) {
    field = {
      ...field,
      name: value.detail,
    };
  }

  function handleTypeChange(value: CustomEvent<string>) {
    field = {
      ...field,
      type: value.detail as DataFieldType,
    };
  }

  function handleOptionsChange(options: string[]) {
    field = {
      ...field,
      typeConfig: {
        ...field.typeConfig,
        options,
      },
    };
  }

  function handleRichTextChange({ detail: richText }: CustomEvent<boolean>) {
    field = {
      ...field,
      typeConfig: {
        ...field.typeConfig,
        richText,
      },
    };
  }

  $: options = [
    { label: $i18n.t("data-types.string"), value: DataFieldType.String },
    { label: $i18n.t("data-types.number"), value: DataFieldType.Number },
    { label: $i18n.t("data-types.boolean"), value: DataFieldType.Boolean },
    { label: $i18n.t("data-types.date"), value: DataFieldType.Date },
    { label: $i18n.t("data-types.unknown"), value: DataFieldType.Unknown },
  ];
</script>

<ModalLayout {title}>
  <ModalContent>
    <SettingItem name={$i18n.t("modals.field.configure.name.name")}>
      <TextInput
        readonly={!editable}
        value={field.name}
        on:input={handleNameChange}
      />
    </SettingItem>
    <SettingItem
      name={$i18n.t("modals.field.configure.type.name")}
      description={$i18n.t("modals.field.configure.type.description")}
    >
      <Select
        disabled
        value={field.type}
        {options}
        on:change={handleTypeChange}
      />
    </SettingItem>
    {#if field.type === DataFieldType.String && !field.repeated && !field.identifier}
      <SettingItem
        name={$i18n.t("modals.field.configure.options.name")}
        description={$i18n.t("modals.field.configure.options.description")}
        vertical
      >
        <MultiTextInput
          options={field.typeConfig?.options ?? []}
          onChange={handleOptionsChange}
        />
      </SettingItem>
      <SettingItem
        name={$i18n.t("modals.field.configure.rich-text.name")}
        description={$i18n.t("modals.field.configure.rich-text.description")}
      >
        <Switch
          checked={field.typeConfig?.richText ?? false}
          on:check={handleRichTextChange}
        />
      </SettingItem>
    {/if}
  </ModalContent>
  <ModalButtonGroup>
    <Button
      variant="primary"
      on:click={() => {
        onSave(field);
      }}>{$i18n.t("modals.field.configure.save")}</Button
    >
  </ModalButtonGroup>
</ModalLayout>
