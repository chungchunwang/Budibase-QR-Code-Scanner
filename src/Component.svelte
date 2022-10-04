<script>
  import { getContext, onMount, onDestroy } from "svelte";
  import { Html5QrcodeScanner } from "html5-qrcode";
  export let field;
  export let label;
  export let autoStartCamera;
  export let fps;
  export let scannerBox;
  export let scannerBoxWidth;
  export let scannerBoxHeight;

  let fieldApi;
  let fieldState; 
  let formStep;
  let formField;

  let html5QrcodeScanner;

  let success = false;

  const { styleable } = getContext("sdk");
  const component = getContext("component");
  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const formApi = formContext?.formApi;
  formStep = formStepContext ? $formStepContext || 1 : 1;
  formField = formApi?.registerField(field, "text", "", false, null, formStep);
  let unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });

  const fieldGroupContext = getContext("field-group");

  const labelPos = fieldGroupContext?.labelPosition || "above";

  $: labelClass =
    labelPos === "above" ? "" : `spectrum-FieldLabel--${labelPos}`;

  let qrCodeValue = "";
  $: fieldApi?.setValue(qrCodeValue);

  const createQRScanner = () => {
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
  <label
    class:hidden={!label}
    for={fieldState?.fieldId}
    class={`spectrum-FieldLabel spectrum-FieldLabel--sizeM spectrum-Form-itemLabel ${labelClass}`}
  >
    {label || " "}
  </label>
  <div class="spectrum-Form-itemField">
    {#if !formContext}
      <div class="placeholder">
        Form components need to be wrapped in a form
      </div>
    {:else}
      <div class="CameraBackground">
        <div id="video" class="Video" />
        {#if success}
          <div style="text-align: center;">
            <p>Scanned Result: {qrCodeValue}</p>
            <button on:click={createQRScanner}>Rescan?</button>
          </div>
        {/if}
      </div>
    {/if}
  </div>
</div>

<style>
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
  .spectrum-FieldLabel--right,
  .spectrum-FieldLabel--left {
    padding-right: var(--spectrum-global-dimension-size-200);
  }
</style>
