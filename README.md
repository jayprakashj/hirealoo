# ServiceFrequency Component

A Vue 3 component built with Shadcn-vue that provides an intuitive interface for configuring service frequency schedules with both predefined options and custom configurations.

## Features

### Core Frequency Options
- **Weekly** - Services occur every week
- **Fortnightly** - Services occur every 2 weeks (default selection)
- **Every 4 weeks** - Services occur monthly
- **None** - No recurring service
- **Custom** - Advanced scheduling with day and time specifications

### Custom Scheduling Features
When "Custom" is selected, the component reveals additional configuration options:

#### Day Selection
- Checkboxes for Monday through Sunday
- Multiple days can be selected simultaneously
- Unchecking updates the schedule configuration

#### Time Window Configuration
- **Specify time window(s)** checkbox
  - When enabled: Input fields for start and end times
  - Supports multiple time ranges (e.g., 06:30-07:30, 15:00-17:00)
  - When disabled: Clears all time window values

#### Exact Time Setting
- **Set exact time** checkbox
  - When enabled: Single time input field (e.g., 13:30)
  - When disabled: Clears the exact time value

## Technical Requirements

### Framework & Dependencies
- **Vue 3** - Modern Vue.js framework
- **Shadcn-vue** - UI component library
- **Tailwind CSS** - Utility-first CSS framework

### Props
- `scheduleConfig` (String) - JSON string containing the current schedule configuration
  - If empty or null, defaults to "Fortnightly"
  - Component manages and updates this configuration internally

### Output
- Displays the complete updated JSON configuration string
- Updates are reflected in real-time as options are modified

## Component Structure

```
ServiceFrequency/
├── Card Container (Shadcn-vue Card)
├── Frequency Options (Radio/Radio Group)
│   ├── Weekly
│   ├── Fortnightly (default)
│   ├── Every 4 weeks
│   ├── None
│   └── Custom
├── Custom Configuration Section (conditionally rendered)
│   ├── Day Selection (Monday-Sunday checkboxes)
│   ├── Time Window Configuration
│   │   ├── Enable/disable checkbox
│   │   └── Time range inputs
│   └── Exact Time Configuration
│       ├── Enable/disable checkbox
│       └── Time input field
└── JSON Output Display
    └── Current configuration string
```

## Usage

### Basic Implementation
```vue
<template>
  <ServiceFrequency 
    :scheduleConfig="scheduleConfig"
    @update:scheduleConfig="handleScheduleUpdate"
  />
</template>

<script setup>
import ServiceFrequency from './components/ServiceFrequency.vue'
import { ref } from 'vue'

const scheduleConfig = ref('')
const handleScheduleUpdate = (newConfig) => {
  scheduleConfig.value = newConfig
}
</script>
```

### Props
| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `scheduleConfig` | String | Yes | `''` | JSON string containing schedule configuration |

### Events
| Event | Payload | Description |
|-------|---------|-------------|
| `update:scheduleConfig` | String | Emitted when schedule configuration changes |

## Design Principles

### User Experience
- **Clean Interface**: Minimalist design with clear visual hierarchy
- **Progressive Disclosure**: Custom options only appear when needed
- **Real-time Feedback**: Immediate updates to configuration display
- **Intuitive Controls**: Familiar checkbox and radio button patterns

### Layout & Spacing
- Consistent spacing between sections
- Clear visual separation of different configuration areas
- Responsive design that works across different screen sizes
- Proper alignment and grouping of related controls

### Accessibility
- Semantic HTML structure
- Proper labeling for form controls
- Keyboard navigation support
- Screen reader compatibility

## JSON Configuration Format

The component expects and produces a JSON string with the following structure:

```json
{
  "frequency": "custom",
  "days": ["monday", "wednesday", "friday"],
  "timeWindows": [
    {
      "start": "06:30",
      "end": "07:30"
    },
    {
      "start": "15:00",
      "end": "17:00"
    }
  ],
  "exactTime": "13:30"
}
```

## Development

### Project Structure
```
src/
├── components/
│   ├── ServiceFrequency.vue
│   └── ui/           # Shadcn-vue components
├── lib/
│   └── utils.js      # Utility functions
├── App.vue           # Main application
└── main.js          # Application entry point
```

### Building
```bash
# Install dependencies
npm install

# Development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

## Browser Support

- Modern browsers with ES6+ support
- Chrome 60+
- Firefox 55+
- Safari 12+
- Edge 79+

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

[Add your license information here]

## Support

For questions or issues, please [create an issue](link-to-issues) or contact the development team.
