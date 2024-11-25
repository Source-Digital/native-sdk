# SourceSync SDK

## Overview
SourceSync is an SDK with Android, C#, Swift, Kotlin, Web, React-Native, and BrightScript (Roku) native implementations.

SourceSync allows synchronized overlay content to be displayed on top of video content. The system supports multiple overlay positions with independent content streams, making it possible to show different content at different positions simultaneously.

## Core Components

### SourceSync
Handles:
- Initialization and setup
- Distribution loading and management
- Overlay creation and positioning
- Video position monitoring
- Activation state management
- Content display synchronization

Key methods:
```java
// Initialize the SDK
SourceSync sourcesync = SourceSync.setup(context, "api-key");

// Load a distribution
Distribution distribution = sourcesync.getDistribution("distribution-id");

// Create overlay containers for specified positions
Map<String, View> overlays = sourcesync.createPositionedOverlays(
    distribution, 
    videoView, 
    "top", "bottom"
);
```

### Distribution
Handles:
- Distribution metadata
- List of activation instances
- Time window definitions

Structure:
```java
Distribution
├── id
├── name
└── activations[]
    ├── externalId
    └── timeWindows[]
        ├── startTime
        ├── endTime
        ├── settings
        └── position
```

### Activation
Handles:
- Content metadata
- Display settings
- Template definition
- Visual content data

### SourceSyncView
It:
- Processes content templates
- Manages visual components
- Handles layout and positioning
- Supports different content types through segment processors

## Content Flow

1. **Distribution Loading**
   - SDK loads distribution JSON from assets or URLs
   - Parses activation references
   - Pre-caches activation content

2. **Overlay Creation**
   - Creates containers for each specified position
   - Initializes SourceSyncView instances
   - Sets up position-specific layouts

3. **Playback Monitoring**
   - Monitors video position at 100ms intervals
   - Identifies active time windows
   - Triggers content updates as needed

4. **Content Display**
   - Loads activation content when needed
   - Updates relevant overlay positions
   - Manages visibility states
   - Logs state changes

## Rendering content natively

While SourceSync of course supports web, many platforms like AppleTV and Roku never will, by design. Therefore, you'll reach a wider audience on CTV and a growing number of specific ad platforms if you use native rendering instead of web.

The problem is, there is no widely accepted standard for rendering natively across all platforms.

However, most design tools and AI settled on "segmented strings", especally a JSON representation of them.

This is what SourceSync uses as well.

## What is a segmented string?

The classical and simple "segmented string" pattern is used throughout computer science to render UI systems very quickly on native devices with little compute power.

Everything from your OS terminal, to your browser console, to Figma, to Adobe XD and more all use the "segmented string" render pattern.

All native platforms, including iOS and Android also use segmented strings. This is because the pattern is extremely fast for hardware to understand and draw.

iOS, Android, AppleTV, Google TV, Roku and other devices all use segmented strings under the hood to draw all UI at the operating system level.

## What does a segmented string look like?

When it's ready to be rendered by your OS, a segmented string looks much like this (iOS and Android both modify this a little):
```json
[{segment:"text",content:"hello world!"}]
```

However, most segmented strings are stored as JSON till they are ready to be rendered, so that they are more human-readable and easier to reason with and edit directly. We do this too, and that just looks like this:

```json
[
  {
    "type": "text",
    "content": "hello world!"
  }
]
```

If you've ever looked at Figma or Adobe XD files, their format is extremely similar, in fact we designed out system to be interoperable with Figma and Adobe products!

### Segment Processors

SourceSync uses a processor system for different segment types. The following exist today, and more will be added with each additional release:

- `text`: Text content
- `image`: Image content
- `button`: Interactive buttons
- `row`: Horizontal layouts
- `column`: Vertical layouts

Custom segment types can be added by:

1. Implementing the `SegmentProcessor` interface
2. Registering the processor with `SegmentProcessorFactory`

Pull requests are always welcome!

## State Management

The SDK maintains several state maps:
```java
// Track overlay views by position
private final Map<String, SourceSyncView> overlayViews

// Track current activation IDs by position
private final Map<String, Integer> currentActivationIds

// Cache activation content
private final Map<Integer, Activation> activationCache
```

## Distribution JSON Format
```json
{
  "id": "unique-id",
  "name": "distribution-name",
  "data": [
    {
      "externalId": 1,
      "instances": [
        {
          "startTime": 1000,
          "endTime": 5000,
          "settings": {
            "position": "top"
          }
        }
      ]
    }
  ]
}
```

## Activation JSON Format
```json
{
  "id": "activation-id",
  "name": "activation-name",
  "settings": {
    "preview": {
      "template": [...]
    }
  },
  "template": [
    {
      "name": "NativeBlock",
      "settings": {
        "segments": [...]
      }
    }
  ]
}
```

## Adding New Features

When extending the SDK:

1. **New Position Types**
   - Add position handling in `createPositionedOverlays`
   - Update layout parameters as needed
   - Consider adding position-specific behaviors

2. **New Content Types**
   - Create new segment processor
   - Implement rendering logic
   - Register with SegmentProcessorFactory

3. **New Features**
   - Consider state management implications
   - Update relevant monitoring loops
   - Add appropriate logging
   - Maintain backwards compatibility

## Best Practices

1. **State Management**
   - Always track state changes
   - Update state atomically
   - Clear state appropriately

2. **Error Handling**
   - Log errors with context
   - Fail gracefully
   - Maintain video playback

3. **Performance**
   - Cache expensive operations
   - Minimize UI updates
   - Consider memory usage

4. **Logging**
   - Log state transitions
   - Include timing information
   - Log errors with stack traces

## Common Tasks

### Adding a New Position Type
```java
// In createPositionedOverlays
FrameLayout container = new FrameLayout(context);
LayoutParams params = new LayoutParams(
    LayoutParams.MATCH_PARENT,
    LayoutParams.WRAP_CONTENT
);
params.gravity = Gravity.YOUR_POSITION;
container.setLayoutParams(params);
```

### Adding a New Segment Type
```java
public class NewSegmentProcessor implements SegmentProcessor {
    @Override
    public View processSegment(Context context, JSONObject segment) {
        // Implementation
    }

    @Override
    public String getSegmentType() {
        return "new-type";
    }
}

// Register in SegmentProcessorFactory
registerProcessor(new NewSegmentProcessor());
```

### Adding New Settings
```java
public class TimeWindow {
    // Add new field
    public final String newSetting;

    private TimeWindow(...) {
        // Parse from settings
        this.newSetting = settings.optString("newSetting", "default");
    }
}
```

## Testing

When testing new features:
1. Test multiple positions simultaneously
2. Verify state transitions
3. Check edge cases in timing
4. Validate error handling
5. Monitor performance impact
6. Test with various content types
7. Verify backwards compatibility

## Future Considerations

Potential enhancements:
1. Animation support
2. Position-specific styling
3. Configurable update intervals
4. Manual position control
5. Enhanced error handling
6. Content preloading
7. Performance optimizations
8. Transition effects

## Logging

The SDK uses your native logging system respectively, i.e. for Java, logs are prefixed with the tag "SourceSync":
```java
Log.d(TAG, String.format("Showing activation %d at position %s, time %d",
    activationId, position, currentPosition));
Log.d(TAG, String.format("Hiding activation %d at position %s, time %d",
    activationId, position, currentPosition));
```

Monitor logs to track:
- Activation changes
- Content loading
- State transitions
- Error conditions