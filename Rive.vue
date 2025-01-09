<template>
  <div class="rive-container">
    <canvas ref="canvas" width="400" height="400"></canvas>
    <div class="debug-info">
      <h3>Debug Information:</h3>
      <p>Current State: {{ currentState }}</p>
      <p>Error: {{ error }}</p>
      <p>File Status: {{ fileStatus }}</p>
    </div>
    <div class="controls">
      <button @click="setState(0)">Idle</button>
      <button @click="setState(1)">Success</button>
      <button @click="setState(2)">Error</button>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import { Rive, Layout } from '@rive-app/canvas';

export default {
  name: 'RivePlayer',
  setup() {
    const canvas = ref(null);
    const error = ref(null);
    const currentState = ref(null);
    const fileStatus = ref('Not loaded');

    let riveInstance = null;
    let stateMachineInput = null;

    const log = (message, data) => {
      console.log(`[Rive Debug] ${message}:`, data);
      fileStatus.value = message;
    };

    const setState = (state) => {
      try {
        if (!stateMachineInput) {
          throw new Error('State machine not initialized');
        }

        log('Setting state to', state);
        stateMachineInput.value = state;
        currentState.value = state;
        error.value = null;
      } catch (e) {
        error.value = e.message;
        console.error('[Rive Error]', e);
      }
    };

    onMounted(() => {
      try {
        log('Mounting component');

        riveInstance = new Rive({
          canvas: canvas.value,
          src: 'https://raw.githubusercontent.com/nameisxi/rive-test-repo/557d15883a19802699e6443e41c444848e9515c8/google-connect.riv',
          stateMachines: 'State Machine 1',
          layout: new Layout({
            fit: 'contain',
            alignment: 'center',
          }),
          autoplay: true,
          onLoad: () => {
            log('File loaded successfully');

            // Get all inputs for the state machine
            const inputs = riveInstance.stateMachineInputs('State Machine 1');
            log('Available inputs', inputs);

            // Find the state input (likely named 'state' or similar)
            stateMachineInput = inputs.find(
              (input) =>
                input.name.toLowerCase().includes('state') ||
                input.type === 'number'
            );

            if (stateMachineInput) {
              log('Found state input', stateMachineInput.name);
              // Set initial state to idle (0)
              setState(0);
            } else {
              throw new Error('Could not find state input');
            }
          },
          onError: (err) => {
            error.value = `Load error: ${err}`;
            log('Load error', err);
          },
        });
      } catch (e) {
        error.value = `Init error: ${e.message}`;
        log('Initialization error', e);
      }
    });

    onBeforeUnmount(() => {
      if (riveInstance) {
        riveInstance.cleanup();
        log('Cleanup complete');
      }
    });

    return {
      canvas,
      error,
      currentState,
      fileStatus,
      setState,
    };
  },
};
</script>

<style scoped>
.rive-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
}

canvas {
  border: 1px solid #ccc;
  background: #f5f5f5;
}

.debug-info {
  margin: 1rem;
  padding: 1rem;
  background: #f8f9fa;
  border: 1px solid #dee2e6;
  border-radius: 4px;
  width: 100%;
  max-width: 600px;
}

.debug-info p {
  margin: 0.5rem 0;
  font-family: monospace;
}

.controls {
  display: flex;
  gap: 0.5rem;
}

button {
  padding: 0.5rem 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  cursor: pointer;
  background-color: white;
  transition: all 0.2s ease;
}

button:hover {
  background-color: #f0f0f0;
  transform: translateY(-1px);
}

button:active {
  transform: translateY(0);
}
</style>
