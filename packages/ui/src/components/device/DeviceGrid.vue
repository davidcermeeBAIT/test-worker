<template>
  <div class="device-grid-container">
    <!-- Settings Panel -->
    <v-expansion-panels class="mb-6">
      <v-expansion-panel>
        <v-expansion-panel-title>
          <v-icon icon="mdi-cog" class="mr-2" />
          Stream Settings
        </v-expansion-panel-title>
        <v-expansion-panel-text>
          <v-row>
            <v-col cols="12" sm="6" md="3">
              <v-select
                v-model="streamSettings.audioEncoder"
                :items="adbStore.audioEncoders"
                item-value="id"
                :item-title="(item) => `${item.codec}${!['off', 'raw'].includes(item.codec) ? ` - ${item.name}` : ''}`"
                label="Audio Encoder"
                required
              />
            </v-col>
            <v-col cols="12" sm="6" md="3">
              <v-select
                v-model="streamSettings.videoEncoder"
                :items="adbStore.videoEncoders"
                item-value="id"
                :item-title="(item) => `${item.codec}${!['off'].includes(item.codec) ? `- ${item.name} (${item.decoder})` : ''}`"
                label="Video Encoder"
                required
              />
            </v-col>
            <v-col cols="12" sm="6" md="3">
              <v-number-input
                v-model="streamSettings.maxFps"
                label="Max FPS"
                :max="90"
                :min="1"
                control-variant="stacked"
              />
            </v-col>
            <v-col cols="12" sm="6" md="3">
              <v-number-input
                v-model="streamSettings.bitRate"
                label="Bit Rate (Mbps)"
                :max="16"
                :min="1"
                control-variant="stacked"
              />
            </v-col>
          </v-row>
        </v-expansion-panel-text>
      </v-expansion-panel>
    </v-expansion-panels>

    <!-- Control Buttons -->
    <v-row class="d-flex align-center justify-center mb-6">
      <v-col cols="auto">
        <v-btn
          color="primary"
          size="large"
          variant="flat"
          @click="startAllStreams"
          :loading="isStartingAll"
          :disabled="!adbStore.devices.length"
        >
          <v-icon icon="mdi-play-circle" start />
          Start All Streams
        </v-btn>
      </v-col>
      <v-col cols="auto">
        <v-btn
          color="error"
          size="large"
          variant="flat"
          @click="stopAllStreams"
          :disabled="!activeStreams.length"
        >
          <v-icon icon="mdi-stop-circle" start />
          Stop All Streams
        </v-btn>
      </v-col>
      <v-col cols="auto">
        <v-btn
          color="warning"
          size="large"
          variant="flat"
          @click="refreshDevices"
          :loading="isRefreshing"
        >
          <v-icon icon="mdi-refresh" start />
          Refresh Devices
        </v-btn>
      </v-col>
      <v-col cols="auto">
        <v-btn
          color="info"
          size="large"
          variant="outlined"
          @click="checkConnections"
        >
          <v-icon icon="mdi-connection" start />
          Check Connections
        </v-btn>
      </v-col>
      <v-col cols="auto">
        <v-btn
          color="secondary"
          size="large"
          variant="outlined"
          @click="testEncoders"
        >
          <v-icon icon="mdi-test-tube" start />
          Test Encoders
        </v-btn>
      </v-col>
    </v-row>

    <!-- Device Grid -->
    <v-row>
      <v-col
        v-for="device in adbStore.devices"
        :key="device.serial"
        cols="12"
        sm="6"
        md="4"
        lg="3"
        xl="2"
      >
        <v-card
          class="device-card"
          :class="{ 'streaming': isDeviceStreaming(device.serial) }"
          elevation="3"
          hover
        >
          <v-card-title class="text-subtitle-1 font-weight-bold pa-3">
            <div class="d-flex align-center justify-space-between w-100">
              <div class="d-flex align-center">
                <v-icon icon="mdi-cellphone-link" class="mr-2" />
                {{ device.serial }}
              </div>
              <v-chip
                :color="getDeviceStatusColor(device)"
                size="small"
                label
              >
                {{ getDeviceStatusText(device) }}
              </v-chip>
            </div>
          </v-card-title>

          <v-card-text class="pa-3">
            <div class="device-info">
              <div class="info-row">
                <span class="label">Model:</span>
                <span class="value">{{ device.model || 'Unknown' }}</span>
              </div>
              <div class="info-row">
                <span class="label">Android:</span>
                <span class="value">{{ device.android || 'Unknown' }}</span>
              </div>
              <div class="info-row">
                <span class="label">Status:</span>
                <v-chip
                  :color="isDeviceStreaming(device.serial) ? 'success' : 'default'"
                  size="small"
                  label
                >
                  {{ isDeviceStreaming(device.serial) ? 'Streaming' : 'Idle' }}
                </v-chip>
              </div>
              <div v-if="isDeviceStreaming(device.serial)" class="info-row">
                <span class="label">Quality:</span>
                <span class="value">{{ streamSettings.maxFps }}fps / {{ streamSettings.bitRate }}Mbps</span>
              </div>
            </div>
          </v-card-text>

          <v-card-actions class="pa-3">
            <v-btn
              v-if="!isDeviceStreaming(device.serial)"
              color="primary"
              variant="flat"
              size="small"
              @click="startDeviceStream(device)"
              :loading="startingDevices.has(device.serial)"
            >
              <v-icon icon="mdi-play" start />
              Start
            </v-btn>
            <v-btn
              v-else
              color="error"
              variant="flat"
              size="small"
              @click="stopDeviceStream(device.serial)"
            >
              <v-icon icon="mdi-stop" start />
              Stop
            </v-btn>
            <v-spacer />
            <v-btn
              icon="mdi-fullscreen"
              size="small"
              variant="text"
              @click="fullscreenDevice(device.serial)"
              :disabled="!isDeviceStreaming(device.serial)"
            />
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>

    <!-- No Devices Message -->
    <v-row v-if="!adbStore.devices.length" class="mt-8">
      <v-col cols="12" class="text-center">
        <v-icon icon="mdi-cellphone-off" size="64" color="grey" />
        <h3 class="text-h5 mt-4 text-grey">No Devices Found</h3>
        <p class="text-body-1 text-grey">Connect your Android devices to get started.</p>
        <v-btn
          color="primary"
          variant="outlined"
          @click="refreshDevices"
          class="mt-4"
        >
          <v-icon icon="mdi-refresh" start />
          Refresh Devices
        </v-btn>
      </v-col>
    </v-row>

    <!-- Streaming Grid View -->
    <div class="streaming-grid mt-8">
      <v-divider class="mb-4" />
      <div class="d-flex align-center justify-between mb-4">
        <h3 class="text-h5 mb-0">Active Streams ({{ activeStreams.length }})</h3>
        <div class="d-flex align-center gap-2">
          <v-chip color="success" label>
            {{ activeStreams.length }} device{{ activeStreams.length !== 1 ? 's' : '' }} streaming
          </v-chip>
          <v-btn
            size="small"
            variant="outlined"
            @click="toggleStreamLayout"
          >
            <v-icon :icon="streamLayout === 'grid' ? 'mdi-view-list' : 'mdi-view-grid'" />
            {{ streamLayout === 'grid' ? 'List' : 'Grid' }}
          </v-btn>
        </div>
      </div>
      
      <!-- No Active Streams Message -->
      <div v-if="!activeStreams.length" class="text-center py-8">
        <v-icon icon="mdi-video-off" size="64" color="grey" />
        <h4 class="text-h6 mt-4 text-grey">No Active Streams</h4>
        <p class="text-body-2 text-grey">Start streaming from devices above to see them here.</p>
      </div>
      
      <!-- Grid Layout -->
      <v-row v-if="streamLayout === 'grid' && activeStreams.length">
        <v-col
          v-for="stream in activeStreams"
          :key="stream.deviceSerial"
          cols="12"
          sm="6"
          md="4"
          lg="3"
          xl="2"
        >
          <div class="stream-container">
            <div class="stream-header">
              <span class="device-name">{{ stream.deviceSerial }}</span>
              <div class="stream-controls">
                <v-btn
                  icon="mdi-fullscreen"
                  size="small"
                  variant="text"
                  @click="fullscreenDevice(stream.deviceSerial)"
                />
                <v-btn
                  icon="mdi-close"
                  size="small"
                  variant="text"
                  @click="stopDeviceStream(stream.deviceSerial)"
                  color="error"
                />
              </div>
            </div>
            <div
              :id="`stream-${stream.deviceSerial}`"
              class="stream-video"
              :style="{
                width: '100%',
                height: '200px',
                background: 'black',
                borderRadius: '8px',
                overflow: 'hidden'
              }"
            />
            <div class="stream-footer">
              <v-chip size="small" color="success" label>
                <v-icon icon="mdi-video" size="small" start />
                Live
              </v-chip>
              <v-chip size="small" color="info" label class="ml-2">
                {{ streamSettings.maxFps }}fps
              </v-chip>
            </div>
          </div>
        </v-col>
      </v-row>

      <!-- List Layout -->
      <div v-else-if="streamLayout === 'list' && activeStreams.length" class="stream-list">
        <v-card
          v-for="stream in activeStreams"
          :key="stream.deviceSerial"
          class="stream-list-item mb-3"
          elevation="2"
        >
          <v-card-title class="text-subtitle-1 pa-3">
            <div class="d-flex align-center justify-space-between w-100">
              <span>{{ stream.deviceSerial }}</span>
              <div class="d-flex align-center gap-2">
                <v-chip size="small" color="success" label>
                  <v-icon icon="mdi-video" size="small" start />
                  Live
                </v-chip>
                <v-btn
                  icon="mdi-fullscreen"
                  size="small"
                  variant="text"
                  @click="fullscreenDevice(stream.deviceSerial)"
                />
                <v-btn
                  icon="mdi-close"
                  size="small"
                  variant="text"
                  @click="stopDeviceStream(stream.deviceSerial)"
                  color="error"
                />
              </div>
            </div>
          </v-card-title>
          <v-card-text class="pa-3">
            <div
              :id="`stream-${stream.deviceSerial}`"
              class="stream-video-list"
              :style="{
                width: '100%',
                height: '300px',
                background: 'black',
                borderRadius: '8px',
                overflow: 'hidden'
              }"
            />
          </v-card-text>
        </v-card>
      </div>
    </div>

    <!-- Success/Error Messages -->
    <v-snackbar
      v-model="showSnackbar"
      :color="snackbarColor"
      :timeout="3000"
      location="top"
    >
      {{ snackbarMessage }}
      <template v-slot:actions>
        <v-btn
          color="white"
          variant="text"
          @click="showSnackbar = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useAdbStore } from '@/store/adb'
import { useToastStore } from '@/store/toast'
import { streamingService } from '@/services/stream/streaming-service'
import { DEAULT_BIT_RATE, DEAULT_MAX_FPS } from '@/utils/constants'
import {
  ScrcpyVideoCodecId,
} from '@yume-chan/scrcpy'
import { TinyH264Decoder } from '@yume-chan/scrcpy-decoder-tinyh264'
import { WebCodecsVideoDecoder } from '@yume-chan/scrcpy-decoder-webcodecs'
import { ReadableStream, WritableStream } from '@yume-chan/stream-extra'
import { Packr, Unpackr } from 'msgpackr'
import { PACK_OPTIONS } from '@/utils/constants'

const adbStore = useAdbStore()
const toastStore = useToastStore()

const activeStreams = ref([])
const startingDevices = ref(new Set())
const isStartingAll = ref(false)
const isRefreshing = ref(false)
const streamLayout = ref('grid') // 'grid' or 'list'

// Connection health monitoring
const connectionHealth = ref(new Map())
let healthCheckInterval = null

// Snackbar state
const showSnackbar = ref(false)
const snackbarMessage = ref('')
const snackbarColor = ref('success')

// Stream settings
const streamSettings = ref({
  audioEncoder: adbStore.audioEncoder || 'raw',
  videoEncoder: adbStore.videoEncoder || 'WebCodecs@h264', // Changed from TinyH264 to WebCodecs for better compatibility
  maxFps: DEAULT_MAX_FPS,
  bitRate: DEAULT_BIT_RATE,
})

const packer = new Packr(PACK_OPTIONS)
const unpacker = new Unpackr(PACK_OPTIONS)

const showMessage = (message, color = 'success') => {
  snackbarMessage.value = message
  snackbarColor.value = color
  showSnackbar.value = true
}

const startHealthCheck = () => {
  if (healthCheckInterval) {
    clearInterval(healthCheckInterval)
  }
  
  healthCheckInterval = setInterval(() => {
    activeStreams.value.forEach(stream => {
      if (stream.ws) {
        const state = stream.ws.readyState
        const lastState = connectionHealth.value.get(stream.deviceSerial)?.state
        
        if (state !== lastState) {
          console.log(`Device ${stream.deviceSerial} WebSocket state changed: ${lastState} -> ${state}`)
          connectionHealth.value.set(stream.deviceSerial, {
            state,
            lastCheck: Date.now(),
            url: stream.ws.url
          })
          
          // Check for unexpected disconnections
          if (state === WebSocket.CLOSED && lastState === WebSocket.OPEN) {
            console.warn(`Device ${stream.deviceSerial} WebSocket closed unexpectedly`)
            showMessage(`Device ${stream.deviceSerial} connection lost`, 'warning')
          }
        }
      }
    })
  }, 2000) // Check every 2 seconds
}

const stopHealthCheck = () => {
  if (healthCheckInterval) {
    clearInterval(healthCheckInterval)
    healthCheckInterval = null
  }
}

const isDeviceStreaming = (deviceSerial) => {
  return activeStreams.value.some(stream => stream.deviceSerial === deviceSerial)
}

const getDeviceStatusColor = (device) => {
  if (isDeviceStreaming(device.serial)) return 'success'
  if (device.status === 'online') return 'info'
  if (device.status === 'offline') return 'error'
  return 'default'
}

const getDeviceStatusText = (device) => {
  if (isDeviceStreaming(device.serial)) return 'Streaming'
  if (device.status === 'online') return 'Online'
  if (device.status === 'offline') return 'Offline'
  return 'Unknown'
}

const toggleStreamLayout = () => {
  streamLayout.value = streamLayout.value === 'grid' ? 'list' : 'grid'
}

const fullscreenDevice = (deviceSerial) => {
  const stream = activeStreams.value.find(s => s.deviceSerial === deviceSerial)
  if (stream && stream.renderer) {
    try {
      if (stream.renderer.requestFullscreen) {
        stream.renderer.requestFullscreen()
      } else if (stream.renderer.webkitRequestFullscreen) {
        stream.renderer.webkitRequestFullscreen()
      } else if (stream.renderer.msRequestFullscreen) {
        stream.renderer.msRequestFullscreen()
      }
      showMessage(`Device ${deviceSerial} entered fullscreen mode`)
    } catch (error) {
      console.error('Fullscreen error:', error)
      showMessage('Failed to enter fullscreen mode', 'error')
    }
  }
}

const checkEncoderCompatibility = (device) => {
  console.log(`=== Encoder Compatibility Check for ${device.serial} ===`)
  
  if (adbStore.audioEncoders) {
    console.log('Available Audio Encoders:')
    adbStore.audioEncoders.forEach(encoder => {
      console.log(`  - ${encoder.codec}${encoder.name !== 'off' ? ` (${encoder.name})` : ''}`)
    })
  }
  
  if (adbStore.videoEncoders) {
    console.log('Available Video Encoders:')
    adbStore.videoEncoders.forEach(encoder => {
      console.log(`  - ${encoder.codec}${encoder.name !== 'off' ? ` (${encoder.name})` : ''} [${encoder.decoder}]`)
    })
  }
  
  // Suggest best encoder based on device info
  if (device.model && device.model.toLowerCase().includes('samsung')) {
    console.log('⚠️  Samsung device detected - may have encoder compatibility issues')
    console.log('💡  Recommended: Try WebCodecs@h264 first, then TinyH264@h264')
  }
  
  if (device.android && parseInt(device.android) < 8) {
    console.log('⚠️  Older Android version detected - limited encoder support')
    console.log('💡  Recommended: Use basic encoders or disable video if needed')
  }
  
  console.log('=== End Compatibility Check ===')
}

const refreshDevices = async () => {
  isRefreshing.value = true
  try {
    await adbStore.metainfo()
    showMessage(`Refreshed devices. Found ${adbStore.devices.length} device(s)`)
    
    // Check compatibility for each device
    adbStore.devices.forEach(device => {
      checkEncoderCompatibility(device)
    })
  } catch (error) {
    console.error('Failed to refresh devices:', error)
    showMessage('Failed to refresh devices', 'error')
  } finally {
    isRefreshing.value = false
  }
}

const startDeviceStream = async (device) => {
  if (startingDevices.value.has(device.serial)) return
  
  startingDevices.value.add(device.serial)
  
  try {
    const stream = await createDeviceStream(device)
    activeStreams.value.push(stream)
    showMessage(`Started streaming for device ${device.serial}`)
  } catch (error) {
    console.error(`Failed to start stream for device ${device.serial}:`, error)
    
    // Try fallback encoders if the primary one fails
    if (error.message.includes('not supported') || error.message.includes('exited prematurely')) {
      console.log(`Trying fallback encoders for device ${device.serial}`)
      await tryFallbackEncoders(device)
    } else {
      showMessage(`Failed to start stream for ${device.serial}`, 'error')
    }
  } finally {
    startingDevices.value.delete(device.serial)
  }
}

const tryFallbackEncoders = async (device) => {
  const fallbackEncoders = [
    'WebCodecs@h264',
    'TinyH264@h264',
    'WebCodecs@h265',
    'off' // Disable video if all else fails
  ]
  
  for (const fallbackEncoder of fallbackEncoders) {
    if (fallbackEncoder === streamSettings.value.videoEncoder) {
      continue // Skip the one we already tried
    }
    
    try {
      console.log(`Trying fallback encoder: ${fallbackEncoder} for device ${device.serial}`)
      
      // Temporarily change the encoder
      const originalEncoder = streamSettings.value.videoEncoder
      streamSettings.value.videoEncoder = fallbackEncoder
      
      const stream = await createDeviceStream(device)
      activeStreams.value.push(stream)
      
      showMessage(`Started streaming for ${device.serial} with ${fallbackEncoder}`, 'success')
      console.log(`Successfully started stream with fallback encoder: ${fallbackEncoder}`)
      return
      
    } catch (fallbackError) {
      console.log(`Fallback encoder ${fallbackEncoder} failed:`, fallbackError.message)
      // Restore original encoder
      streamSettings.value.videoEncoder = originalEncoder
      continue
    }
  }
  
  // If all fallbacks fail
  showMessage(`All video encoders failed for ${device.serial}`, 'error')
  console.error(`All fallback encoders failed for device ${device.serial}`)
}

const stopDeviceStream = (deviceSerial) => {
  const streamIndex = activeStreams.value.findIndex(s => s.deviceSerial === deviceSerial)
  if (streamIndex !== -1) {
    const stream = activeStreams.value[streamIndex]
    if (stream.abortController) {
      stream.abortController.abort()
    }
    if (stream.decoder) {
      stream.decoder.dispose()
    }
    if (stream.ws) {
      stream.ws.close()
    }
    
    // Clean up the DOM elements
    if (stream.container) {
      // Find and remove the entire column containing this stream
      const column = stream.container.closest('.v-col')
      if (column) {
        column.remove()
      } else {
        // Fallback: just remove the container
        stream.container.remove()
      }
    }
    
    activeStreams.value.splice(streamIndex, 1)
    showMessage(`Stopped streaming for device ${deviceSerial}`)
  }
}

const startAllStreams = async () => {
  if (isStartingAll.value) return
  
  isStartingAll.value = true
  
  try {
    const promises = adbStore.devices.map(device => startDeviceStream(device))
    await Promise.all(promises)
    showMessage(`Started streaming for all ${adbStore.devices.length} devices`)
  } catch (error) {
    console.error('Failed to start all streams:', error)
    showMessage('Failed to start all streams', 'error')
  } finally {
    isStartingAll.value = false
  }
}

const stopAllStreams = () => {
  const count = activeStreams.value.length
  activeStreams.value.forEach(stream => {
    if (stream.abortController) {
      stream.abortController.abort()
    }
    if (stream.decoder) {
      stream.decoder.dispose()
    }
    if (stream.ws) {
      stream.ws.close()
    }
  })
  activeStreams.value = []
  showMessage(`Stopped streaming for all ${count} devices`)
}

const createDeviceStream = async (device) => {
  // Create a unique container for this device's stream
  const containerId = `stream-${device.serial}`
  
  // Wait for the container to be available in the DOM
  let container = document.getElementById(containerId)
  let attempts = 0
  const maxAttempts = 10
  
  while (!container && attempts < maxAttempts) {
    await new Promise(resolve => setTimeout(resolve, 100)) // Wait 100ms
    container = document.getElementById(containerId)
    attempts++
  }
  
  if (!container) {
    // If container still doesn't exist, create it dynamically
    console.log(`Creating container dynamically for device ${device.serial}`)
    container = document.createElement('div')
    container.id = containerId
    container.className = 'stream-video'
    container.style.cssText = `
      width: 100%;
      height: 200px;
      background: black;
      border-radius: 8px;
      overflow: hidden;
    `
    
    // Find the streaming grid and add the container
    const streamingGrid = document.querySelector('.streaming-grid')
    if (streamingGrid) {
      // Create a new column for this stream
      const col = document.createElement('div')
      col.className = 'v-col col-12 col-sm-6 col-md-4 col-lg-3 col-xl-2'
      
      const streamContainer = document.createElement('div')
      streamContainer.className = 'stream-container'
      
      const header = document.createElement('div')
      header.className = 'stream-header'
      header.innerHTML = `
        <span class="device-name">${device.serial}</span>
        <div class="stream-controls">
          <button class="v-btn v-btn--icon v-btn--size-small v-btn--variant-text" id="fullscreen-${device.serial}">
            <i class="mdi mdi-fullscreen"></i>
          </button>
          <button class="v-btn v-btn--icon v-btn--size-small v-btn--variant-text v-btn--color-error" id="close-${device.serial}">
            <i class="mdi mdi-close"></i>
          </button>
        </div>
      `
      
      const footer = document.createElement('div')
      footer.className = 'stream-footer'
      footer.innerHTML = `
        <span class="v-chip v-chip--size-small v-chip--color-success v-chip--label">
          <i class="mdi mdi-video"></i>
          Live
        </span>
        <span class="v-chip v-chip--size-small v-chip--color-info v-chip--label ml-2">
          ${streamSettings.value.maxFps}fps
        </span>
      `
      
      streamContainer.appendChild(header)
      streamContainer.appendChild(container)
      streamContainer.appendChild(footer)
      col.appendChild(streamContainer)
      
      // Add to the grid
      const gridRow = streamingGrid.querySelector('.v-row')
      if (gridRow) {
        gridRow.appendChild(col)
      } else {
        // If no grid row exists, create one
        const newRow = document.createElement('div')
        newRow.className = 'v-row'
        newRow.appendChild(col)
        streamingGrid.appendChild(newRow)
      }
      
      // Add event listeners after DOM insertion
      setTimeout(() => {
        const fullscreenBtn = document.getElementById(`fullscreen-${device.serial}`)
        const closeBtn = document.getElementById(`close-${device.serial}`)
        
        if (fullscreenBtn) {
          fullscreenBtn.addEventListener('click', () => fullscreenDevice(device.serial))
        }
        
        if (closeBtn) {
          closeBtn.addEventListener('click', () => stopDeviceStream(device.serial))
        }
      }, 100)
    }
  }

  if (!container) {
    throw new Error(`Failed to create or find container for device ${device.serial}`)
  }

  // Create decoder for this device
  let decoder
  if (['off', 'TinyH264'].includes(streamSettings.value.videoEncoder?.includes('TinyH264') ? 'TinyH264' : 'WebCodecs')) {
    decoder = new TinyH264Decoder()
  } else if (['WebCodecs'].includes(streamSettings.value.videoEncoder?.includes('WebCodecs') ? 'WebCodecs' : 'TinyH264')) {
    let codec
    const videoCodec = streamSettings.value.videoEncoder?.split('@')[1] || 'h264'
    switch (videoCodec) {
      case 'h264': {
        codec = ScrcpyVideoCodecId.H264
        break
      }
      case 'h265': {
        codec = ScrcpyVideoCodecId.H265
        break
      }
      case 'av1': {
        codec = ScrcpyVideoCodecId.AV1
        break
      }
    }
    decoder = new WebCodecsVideoDecoder(codec, false)
  }

  if (!decoder) {
    throw new Error('No suitable decoder available')
  }

  // Set up renderer
  const renderer = decoder.renderer
  renderer.style.maxWidth = '100%'
  renderer.style.maxHeight = '100%'
  renderer.style.touchAction = 'none'
  renderer.style.outline = 'none'
  container.appendChild(renderer)

  // Set up abort controller
  const abortController = new AbortController()

  // Create the stream object first
  const stream = {
    deviceSerial: device.serial,
    ws: null, // Will be set below
    container,
    renderer,
    decoder,
    abortController,
    videoController: null, // Will be set by the ReadableStream
  }

  // Set up video stream
  const videoController = new ReadableStream({
    start(controller) {
      // Store controller reference in the stream object
      stream.videoController = controller
    },
  })

  // Pipe video stream to decoder
  videoController
    .pipeTo(decoder.writable, {
      signal: abortController.signal,
    })
    .catch((e) => {
      if (abortController.signal.aborted) {
        return
      }
      console.error(`Video stream error for device ${device.serial}:`, e)
    })

  // Initialize WebSocket connection
  console.log(`=== Creating WebSocket for device ${device.serial} ===`)
  console.log('Stream settings being sent:', {
    device: device.serial,
    audio: streamSettings.value.audioEncoder !== 'off',
    audioCodec: streamSettings.value.audioEncoder === 'off' ? undefined : streamSettings.value.audioEncoder,
    audioEncoder: streamSettings.value.audioEncoder === 'off' ? undefined : streamSettings.value.audioEncoder,
    video: streamSettings.value.videoEncoder !== 'off',
    videoCodec: streamSettings.value.videoEncoder === 'off' ? undefined : streamSettings.value.videoEncoder.split('@')[1],
    videoEncoder: streamSettings.value.videoEncoder === 'off' ? undefined : streamSettings.value.videoEncoder.split('@')[0],
    videoBitRate: streamSettings.value.bitRate,
    maxFps: streamSettings.value.maxFps,
  })
  
  const ws = await streamingService.init({
    device: device.serial,
    audio: streamSettings.value.audioEncoder !== 'off',
    audioCodec:
      streamSettings.value.audioEncoder === 'off'
        ? undefined
        : streamSettings.value.audioEncoder,
    audioEncoder:
      streamSettings.value.audioEncoder === 'off'
        ? undefined
        : streamSettings.value.audioEncoder,
    video: streamSettings.value.videoEncoder !== 'off',
    videoCodec:
      streamSettings.value.videoEncoder === 'off'
        ? undefined
        : streamSettings.value.videoEncoder.split('@')[1],
    videoEncoder:
      streamSettings.value.videoEncoder === 'off'
        ? undefined
        : streamSettings.value.videoEncoder.split('@')[0],
    videoBitRate: streamSettings.value.bitRate,
    maxFps: streamSettings.value.maxFps,
    onopen: (ws, id, evt) => {
      console.log(`Device ${device.serial} stream connected: ID=${id}`)
      console.log('WebSocket readyState:', ws.readyState)
      console.log('WebSocket URL:', ws.url)
      showMessage(`Device ${device.serial} stream connected`, 'success')
    },
    onclose: (ws, id, evt) => {
      console.log(`Device ${device.serial} stream disconnected: ID=${id}`)
      console.log('Close event details:', evt)
      console.log('WebSocket readyState:', ws.readyState)
      console.log('Close code:', evt.code, 'Close reason:', evt.reason)
      
      // Only remove from active streams if it's not a normal closure
      if (evt.code !== 1000 && evt.code !== 1001) {
        const streamIndex = activeStreams.value.findIndex(s => s.deviceSerial === device.serial)
        if (streamIndex !== -1) {
          activeStreams.value.splice(streamIndex, 1)
          showMessage(`Device ${device.serial} stream disconnected unexpectedly`, 'warning')
        }
      } else {
        showMessage(`Device ${device.serial} stream closed normally`, 'info')
      }
    },
    onmessage: (ws, id, evt) => {
      try {
        console.log(`Received message from device ${device.serial}:`, evt.data)
        const record = unpacker.unpack(evt.data)
        console.log('Unpacked record:', record)
        
        if (record.media === 'video' && stream.videoController) {
          console.log(`Enqueuing video packet for device ${device.serial}`)
          stream.videoController.enqueue(record.packet)
        } else if (record.media === 'audio') {
          console.log(`Received audio data for device ${device.serial}`)
        } else if (record.media === 'message') {
          console.log(`Received message for device ${device.serial}:`, record.message)
        }
      } catch (err) {
        console.error(`Error processing message for device ${device.serial}:`, err)
        console.error('Raw message data:', evt.data)
      }
    },
    onerror: (ws, id, evt) => {
      console.error(`Device ${device.serial} stream error: ID=${id}`, evt)
      console.error('WebSocket error details:', evt)
      
      // Check if this is a scrcpy server error
      if (evt.data && typeof evt.data === 'string') {
        if (evt.data.includes('scrcpy server exited') || 
            evt.data.includes('not supported') ||
            evt.data.includes('exited prematurely')) {
          showMessage(`Scrcpy server error for ${device.serial}. Trying fallback encoders...`, 'warning')
          // The error will be caught by the try-catch in startDeviceStream
          return
        }
      }
      
      showMessage(`Stream error for device ${device.serial}`, 'error')
    },
  })

  // Add WebSocket event listeners for additional debugging
  ws.addEventListener('close', (evt) => {
    console.log(`WebSocket close event for device ${device.serial}:`, evt)
  })

  ws.addEventListener('error', (evt) => {
    console.log(`WebSocket error event for device ${device.serial}:`, evt)
  })

  // Check WebSocket state after connection
  console.log(`WebSocket state after init for device ${device.serial}:`, ws.readyState)
  console.log(`WebSocket URL for device ${device.serial}:`, ws.url)

  // Update the stream object with the WebSocket
  stream.ws = ws

  return stream
}

const checkConnections = () => {
  console.log('=== Connection Status Check ===')
  console.log('Active streams:', activeStreams.value.length)
  
  activeStreams.value.forEach(stream => {
    if (stream.ws) {
      const state = stream.ws.readyState
      const stateText = {
        0: 'CONNECTING',
        1: 'OPEN',
        2: 'CLOSING',
        3: 'CLOSED'
      }[state] || 'UNKNOWN'
      
      console.log(`Device ${stream.deviceSerial}:`)
      console.log(`  WebSocket State: ${state} (${stateText})`)
      console.log(`  URL: ${stream.ws.url}`)
      console.log(`  Buffered Amount: ${stream.ws.bufferedAmount}`)
      console.log(`  Protocol: ${stream.ws.protocol}`)
      console.log(`  Extensions: ${stream.ws.extensions}`)
      
      if (stream.decoder) {
        console.log(`  Decoder: ${stream.decoder.constructor.name}`)
      }
      
      if (stream.videoController) {
        console.log(`  Video Controller: Available`)
      } else {
        console.log(`  Video Controller: Not available`)
      }
    } else {
      console.log(`Device ${stream.deviceSerial}: No WebSocket`)
    }
  })
  
  console.log('Connection Health Map:', connectionHealth.value)
  console.log('=== End Connection Check ===')
  
  showMessage(`Connection status logged to console`, 'info')
}

const testEncoders = () => {
  console.log('=== Testing Encoder Compatibility ===')
  const testDevice = adbStore.devices[0] || { serial: 'localhost', model: 'Localhost Device', android: '10.0' }
  
  if (!testDevice) {
    showMessage('No devices found to test encoders.', 'warning')
    return
  }

  const testEncoders = [
    'WebCodecs@h264',
    'TinyH264@h264',
    'WebCodecs@h265',
    'off'
  ]

  const testPromises = testEncoders.map(async (encoder) => {
    try {
      console.log(`Testing encoder: ${encoder} for device ${testDevice.serial}`)
      streamSettings.value.videoEncoder = encoder
      const stream = await createDeviceStream(testDevice)
      activeStreams.value.push(stream)
      showMessage(`Successfully started stream with ${encoder} for ${testDevice.serial}`, 'success')
      console.log(`Successfully started stream with ${encoder} for ${testDevice.serial}`)
    } catch (error) {
      console.error(`Failed to start stream with ${encoder} for ${testDevice.serial}:`, error)
      showMessage(`Failed to start stream with ${encoder} for ${testDevice.serial}: ${error.message}`, 'error')
      console.error(`Failed to start stream with ${encoder} for ${testDevice.serial}:`, error)
    }
  })

  Promise.all(testPromises).then(() => {
    console.log('=== End of Encoder Testing ===')
    showMessage('Encoder testing complete. Check console for results.', 'info')
  }).catch(err => {
    console.error('Error during encoder testing:', err)
    showMessage('Error during encoder testing.', 'error')
  })
}

onMounted(() => {
  // Initialize stream settings with current store values
  streamSettings.value.audioEncoder = adbStore.audioEncoder || 'raw'
  streamSettings.value.videoEncoder = adbStore.videoEncoder || 'WebCodecs@h264' // Changed from TinyH264 to WebCodecs for better compatibility
  
  // Check environment variables
  console.log('=== Environment Check ===')
  console.log('VITE_BACKEND_WS_URL:', import.meta.env.VITE_BACKEND_WS_URL)
  console.log('NODE_ENV:', import.meta.env.NODE_ENV)
  console.log('MODE:', import.meta.env.MODE)
  console.log('=== End Environment Check ===')
  
  // Start connection health monitoring
  startHealthCheck()
})

onUnmounted(() => {
  // Clean up all streams when component unmounts
  stopAllStreams()
  
  // Stop health monitoring
  stopHealthCheck()
})
</script>

<style scoped>
.device-grid-container {
  padding: 20px;
}

.device-card {
  height: 100%;
  transition: all 0.3s ease;
}

.device-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.device-card.streaming {
  border: 2px solid #4caf50;
  background: linear-gradient(135deg, #f8f9fa 0%, #e8f5e8 100%);
}

.device-info {
  font-size: 0.875rem;
}

.info-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.info-row:last-child {
  margin-bottom: 0;
}

.label {
  font-weight: 500;
  color: #666;
}

.value {
  color: #333;
}

.streaming-grid {
  margin-top: 40px;
}

.stream-container {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  overflow: hidden;
  background: #fafafa;
}

.stream-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  background: #f5f5f5;
  border-bottom: 1px solid #e0e0e0;
}

.device-name {
  font-weight: 500;
  font-size: 0.875rem;
}

.stream-controls {
  display: flex;
  gap: 4px;
}

.stream-video {
  position: relative;
  background: #000;
}

.stream-footer {
  padding: 8px 12px;
  background: #f5f5f5;
  border-top: 1px solid #e0e0e0;
  text-align: center;
}

.stream-list {
  margin-top: 20px;
}

.stream-list-item {
  border: 1px solid #e0e0e0;
}

.stream-video-list {
  position: relative;
  background: #000;
}

.gap-2 {
  gap: 8px;
}
</style> 