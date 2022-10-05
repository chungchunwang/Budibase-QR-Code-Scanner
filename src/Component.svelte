<script>
  //By: Chungchun Wang (https://github.com/chungchunwang)
  //To make the plugin fit the look of Budibase, some of this code and the styling is from the Star Rating component (https://github.com/andz-bb/budibase-component-star-rating) referenced in the docs.

  import { getContext, onDestroy, onMount } from "svelte";
  import { Html5QrcodeScanner } from "html5-qrcode";

  //Property Fields
  export let field;
  export let label;
  export let autoStartCamera;
  export let fps;
  export let scannerBox;
  export let scannerBoxWidth;
  export let scannerBoxHeight;

  const { styleable } = getContext("sdk");
  const component = getContext("component");
  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const fieldGroupContext = getContext("field-group");

  let fieldApi;
  let fieldState;

  const formApi = formContext?.formApi;
  const labelPos = fieldGroupContext?.labelPosition || "above";
  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(
    field,
    "text",
    0,
    false,
    null,
    formStep
  );

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  $: labelClass =
    labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });

  let html5QrcodeScanner;

  let success = false;
  let qrCodeValue = "";
  $: fieldApi?.setValue(qrCodeValue);
  const createQRScanner = () => {
    if(!formContext) return;
    success = false;
    qrCodeValue = "";
    html5QrcodeScanner = new Html5QrcodeScanner(
      "video",
      {
        fps: fps,
        qrbox: scannerBox
          ? { width: scannerBoxWidth, height: scannerBoxHeight }
          : null,
        rememberLastUsedCamera: autoStartCamera,
      },
      /* verbose= */ false
    );
    html5QrcodeScanner.render(onScanSuccess);
  };
  onMount(createQRScanner);
  function onScanSuccess(decodedText, decodedResult) {
    qrCodeValue = decodedText;
    html5QrcodeScanner.clear();
    success = true;
  }
</script>

<div class="spectrum-Form-item" use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Form components need to be wrapped in a form</div>
  {:else}
    <label
      class:hidden={!label}
      for={fieldState?.fieldId}
      class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
    >
      {label || " "}
    </label>
    <div class="spectrum-Form-itemField">
      <div class="CameraBackground">
        <div id="video" class="Video" />
        {#if success}
          <div style="text-align: center;">
            <p>Scanned Result: {qrCodeValue}</p>
            <button on:click={createQRScanner}>Rescan?</button>
          </div>
        {/if}
      </div>
    </div>
  {/if}
</div>

<style>
  .placeholder {
    color: var(--spectrum-global-color-gray-600);
  }
  label {
    white-space: nowrap;
  }
  label.hidden {
    padding: 0;
  }
  .spectrum-Form-itemField {
    position: relative;
    width: 100%;
  }
  .error {
    color: var(
      --spectrum-semantic-negative-color-default,
      var(--spectrum-global-color-red-500)
    );
    font-size: var(--spectrum-global-dimension-font-size-75);
    margin-top: var(--spectrum-global-dimension-size-75);
  }
  .spectrum-FieldLabel--right,
  .spectrum-FieldLabel--left {
    padding-right: var(--spectrum-global-dimension-size-200);
  }
</style>
