# Multi-Device Android Streaming

This application now supports streaming from multiple Android devices simultaneously with a beautiful grid-based UI.

## Features

### 🎯 Multi-Device Support
- **Device Grid View**: See all connected Android devices in a responsive grid layout
- **Individual Device Control**: Start/stop streams for individual devices
- **Bulk Operations**: Start or stop all device streams with single buttons
- **Real-time Status**: Visual indicators showing which devices are streaming

### 🎮 Streaming Controls
- **Stream Settings Panel**: Configure audio/video encoders, FPS, and bitrate for all streams
- **Dual Layout Modes**: Switch between grid and list views for active streams
- **Fullscreen Support**: Enter fullscreen mode for any individual device stream
- **Quality Control**: Adjust streaming parameters per device

### 🎨 User Experience
- **Responsive Design**: Works on desktop, tablet, and mobile devices
- **Visual Feedback**: Color-coded status indicators and hover effects
- **Toast Notifications**: Real-time feedback for all operations
- **Error Handling**: Graceful error handling with user-friendly messages

## How to Use

### 1. Switch to Multi-Device View
- Use the toggle buttons at the top to switch between "Single Device" and "All Devices" views
- The "All Devices" view shows the new grid interface

### 2. Configure Stream Settings
- Click the "Stream Settings" expansion panel to configure:
  - **Audio Encoder**: Choose from available audio codecs
  - **Video Encoder**: Select video encoder and decoder combination
  - **Max FPS**: Set maximum frames per second (1-90)
  - **Bit Rate**: Configure video bitrate in Mbps (1-16)

### 3. Start Streaming
- **Individual Device**: Click the "Start" button on any device card
- **All Devices**: Use the "Start All Streams" button to begin streaming from all devices simultaneously
- **Refresh Devices**: Use the "Refresh Devices" button to detect newly connected devices

### 4. Manage Active Streams
- **Grid Layout**: Compact view showing all active streams in a grid
- **List Layout**: Detailed view with larger video previews
- **Fullscreen**: Click the fullscreen button on any stream to enter fullscreen mode
- **Stop Streams**: Stop individual streams or use "Stop All Streams" for bulk operations

### 5. Monitor Status
- **Device Cards**: Show device information, connection status, and streaming state
- **Stream Indicators**: Visual feedback showing which devices are actively streaming
- **Quality Display**: Shows current FPS and bitrate settings for active streams

## Technical Details

### Architecture
- **Independent Streams**: Each device maintains its own WebSocket connection and video decoder
- **Resource Management**: Proper cleanup of decoders, controllers, and connections
- **Error Handling**: Graceful fallback and user notification for connection issues

### Performance
- **Efficient Decoding**: Uses appropriate decoders (TinyH264 or WebCodecs) based on device capabilities
- **Memory Management**: Automatic cleanup of unused resources
- **Connection Pooling**: Efficient WebSocket management for multiple devices

### Compatibility
- **Browser Support**: Works with modern browsers supporting WebRTC and WebCodecs
- **Device Support**: Compatible with Android devices supporting ADB
- **Network**: Requires stable network connection for multiple simultaneous streams

## Troubleshooting

### Common Issues
1. **Devices Not Detected**: Use the "Refresh Devices" button
2. **Stream Won't Start**: Check device connection and try refreshing
3. **Poor Video Quality**: Adjust FPS and bitrate settings
4. **Multiple Streams Lag**: Reduce FPS or bitrate for better performance

### Performance Tips
- Start with lower FPS (15-30) for multiple devices
- Use appropriate bitrate based on network capacity
- Close unnecessary browser tabs to free up resources
- Ensure stable network connection

## Future Enhancements

- **Stream Recording**: Save individual device streams
- **Device Groups**: Organize devices into logical groups
- **Advanced Controls**: Touch input and keyboard mapping for each device
- **Analytics**: Performance metrics and stream statistics
- **Remote Access**: Access streams from other devices on the network

---

For technical support or feature requests, please refer to the main project documentation. 