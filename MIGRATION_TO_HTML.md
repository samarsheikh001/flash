# React to HTML Migration Guide

## Overview

This document provides a comprehensive guide for migrating the Flash application from React to a simple HTML/JavaScript application. The current application is a single-page application (SPA) that uses React 18 with modern browser APIs for USB device communication and firmware flashing.

## Current Architecture Analysis

### Technology Stack
- **Frontend Framework**: React 18.3.1 with hooks
- **Build Tool**: Vite
- **Styling**: Tailwind CSS with PostCSS
- **State Management**: Local React state (useState, useEffect)
- **Module System**: ES Modules (ESM)
- **Key APIs**: WebUSB, File System Access API

### Application Structure
```
/src
├── app/
│   ├── index.jsx        # Main App component
│   └── Flash.jsx        # Core flashing component
├── utils/
│   ├── manager.js       # FlashManager class
│   ├── image.js         # ImageManager class
│   ├── manifest.js      # Manifest fetching/parsing
│   ├── stream.js        # Streaming downloads
│   ├── platform.js      # Platform detection
│   └── progress.js      # Progress utilities
├── assets/             # SVG icons and images
└── main.jsx           # React entry point
```

## Migration Strategy

### Phase 1: Analysis and Planning

1. **Component Mapping**
   - Map React components to HTML sections
   - Identify reusable UI patterns
   - Document state dependencies

2. **State Management Inventory**
   - List all state variables and their flows
   - Identify event handlers and callbacks
   - Map React hooks to vanilla JS patterns

3. **Dependency Analysis**
   - Identify React-specific dependencies to remove
   - List vanilla JS alternatives for React patterns
   - Keep non-React utilities intact

### Phase 2: Project Setup

1. **Create New HTML Structure**
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>Flash</title>
     <link rel="icon" href="/favicon.ico">
     <link rel="stylesheet" href="/styles/main.css">
     <script type="module" src="/js/main.js"></script>
   </head>
   <body>
     <div id="app"></div>
   </body>
   </html>
   ```

2. **Directory Structure**
   ```
   /
   ├── index.html
   ├── js/
   │   ├── main.js
   │   ├── components/
   │   │   ├── flash.js
   │   │   └── ui.js
   │   └── utils/        # Keep existing utils
   ├── styles/
   │   └── main.css
   └── assets/          # Keep existing assets
   ```

### Phase 3: Component Migration

#### 1. Convert JSX to HTML Templates

**React Component (Flash.jsx)**:
```jsx
function Flash({ onClose }) {
  const [step, setStep] = useState(1);
  const [progress, setProgress] = useState(null);
  
  return (
    <div className="flex flex-col h-full">
      <h1 className="text-2xl">Flash Device</h1>
      {step === 1 && <StepOne />}
      {step === 2 && <StepTwo />}
    </div>
  );
}
```

**HTML/JS Equivalent**:
```javascript
// flash.js
export class FlashComponent {
  constructor(container, options = {}) {
    this.container = container;
    this.onClose = options.onClose;
    this.state = {
      step: 1,
      progress: null
    };
    this.render();
  }

  setState(updates) {
    this.state = { ...this.state, ...updates };
    this.render();
  }

  render() {
    this.container.innerHTML = `
      <div class="flex flex-col h-full">
        <h1 class="text-2xl">Flash Device</h1>
        ${this.state.step === 1 ? this.renderStepOne() : ''}
        ${this.state.step === 2 ? this.renderStepTwo() : ''}
      </div>
    `;
    this.attachEventListeners();
  }

  renderStepOne() {
    return '<div>Step 1 content...</div>';
  }

  attachEventListeners() {
    // Add event listeners after render
  }
}
```

#### 2. State Management Pattern

**Create a simple state management system**:
```javascript
// state.js
export class StateManager {
  constructor(initialState = {}) {
    this.state = initialState;
    this.listeners = [];
  }

  subscribe(listener) {
    this.listeners.push(listener);
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  }

  setState(updates) {
    this.state = { ...this.state, ...updates };
    this.listeners.forEach(listener => listener(this.state));
  }

  getState() {
    return this.state;
  }
}
```

#### 3. Event Handling Migration

**React Pattern**:
```jsx
<button onClick={() => handleClick(item)}>Click</button>
```

**HTML Pattern**:
```javascript
// Use data attributes and event delegation
render() {
  return `<button data-action="click" data-item-id="${item.id}">Click</button>`;
}

attachEventListeners() {
  this.container.addEventListener('click', (e) => {
    if (e.target.dataset.action === 'click') {
      const itemId = e.target.dataset.itemId;
      this.handleClick(itemId);
    }
  });
}
```

### Phase 4: Build Process Migration

#### 1. Remove React Dependencies
```bash
npm uninstall react react-dom @vitejs/plugin-react
```

#### 2. Update Vite Configuration
```javascript
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        main: 'index.html',
      },
    },
  },
  server: {
    port: 3000,
  },
});
```

#### 3. Update Package.json
```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "@commaai/qdl": "^0.1.3",
    "@fontsource-variable/inter": "^5.0.21",
    "@fontsource-variable/jetbrains-mono": "^5.0.22",
    "xz-decompress": "^0.2.2"
  }
}
```

### Phase 5: CSS and Styling

1. **Keep Tailwind CSS** - It works perfectly with vanilla HTML
2. **Update PostCSS config** if needed
3. **Import styles in HTML** instead of JSX

### Phase 6: Testing Migration

1. **Update Test Framework**
   - Remove React Testing Library
   - Use vanilla JS testing with Vitest
   - Update test setup

2. **Example Test Migration**:
```javascript
// Before (React)
import { render, screen } from '@testing-library/react';
test('renders flash button', () => {
  render(<App />);
  expect(screen.getByText(/flash/i)).toBeInTheDocument();
});

// After (Vanilla)
import { FlashComponent } from './flash.js';
test('renders flash button', () => {
  const container = document.createElement('div');
  new FlashComponent(container);
  expect(container.querySelector('button')).toBeTruthy();
});
```

## Migration Checklist

### Pre-Migration
- [ ] Backup current codebase
- [ ] Document all React components and their props
- [ ] List all state variables and their usage
- [ ] Identify all event handlers
- [ ] Map component hierarchy

### During Migration
- [ ] Set up new project structure
- [ ] Create HTML template system
- [ ] Implement state management
- [ ] Convert components one by one
- [ ] Migrate event handlers
- [ ] Update build configuration
- [ ] Preserve all utility functions
- [ ] Maintain WebUSB and File System API integration

### Post-Migration
- [ ] Test all functionality
- [ ] Verify USB device communication
- [ ] Check download and progress tracking
- [ ] Validate error handling
- [ ] Performance testing
- [ ] Update documentation
- [ ] Remove unused dependencies

## Key Considerations

### 1. WebUSB API Integration
The WebUSB API integration in `manager.js` should work identically in vanilla JS since it's already using standard Web APIs.

### 2. File System Access API
The `ImageManager` class in `image.js` will continue to work without modification.

### 3. Streaming Downloads
The streaming functionality in `stream.js` is already vanilla JS and requires no changes.

### 4. Progressive Enhancement
Consider implementing the app with progressive enhancement:
- Basic HTML structure that works without JS
- Enhanced functionality when JS is available
- Graceful degradation for unsupported browsers

### 5. Browser Compatibility
Ensure the migration maintains compatibility with browsers that support:
- WebUSB API
- File System Access API
- ES Modules
- Modern CSS features

## Example Migration: Flash Component

Here's a detailed example of migrating the main Flash component:

### Original React Component (simplified)
```jsx
// Flash.jsx
import { useState, useEffect } from 'react';
import { FlashManager } from '../utils/manager';

export function Flash({ onClose }) {
  const [step, setStep] = useState(1);
  const [device, setDevice] = useState(null);
  const [progress, setProgress] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (device) {
      startFlashing();
    }
  }, [device]);

  const startFlashing = async () => {
    const manager = new FlashManager();
    manager.onProgress = setProgress;
    manager.onError = setError;
    await manager.flash(device);
  };

  return (
    <div className="flash-container">
      {error && <div className="error">{error}</div>}
      {progress && <div className="progress">{progress}%</div>}
      <button onClick={onClose}>Close</button>
    </div>
  );
}
```

### Migrated Vanilla JS Component
```javascript
// components/flash.js
import { FlashManager } from '../utils/manager.js';

export class FlashComponent {
  constructor(container, options = {}) {
    this.container = container;
    this.onClose = options.onClose;
    
    // Initialize state
    this.state = {
      step: 1,
      device: null,
      progress: null,
      error: null
    };
    
    this.manager = new FlashManager();
    this.setupManager();
    this.render();
  }

  setupManager() {
    this.manager.onProgress = (progress) => {
      this.setState({ progress });
    };
    
    this.manager.onError = (error) => {
      this.setState({ error });
    };
  }

  setState(updates) {
    this.state = { ...this.state, ...updates };
    this.render();
    
    // Handle side effects
    if (updates.device && !this.state.device) {
      this.startFlashing();
    }
  }

  async startFlashing() {
    try {
      await this.manager.flash(this.state.device);
    } catch (error) {
      this.setState({ error: error.message });
    }
  }

  render() {
    this.container.innerHTML = `
      <div class="flash-container">
        ${this.state.error ? `<div class="error">${this.state.error}</div>` : ''}
        ${this.state.progress !== null ? `<div class="progress">${this.state.progress}%</div>` : ''}
        <button data-action="close">Close</button>
      </div>
    `;
    
    this.attachEventListeners();
  }

  attachEventListeners() {
    const closeBtn = this.container.querySelector('[data-action="close"]');
    if (closeBtn) {
      closeBtn.addEventListener('click', () => {
        if (this.onClose) this.onClose();
      });
    }
  }

  destroy() {
    // Cleanup
    this.container.innerHTML = '';
    if (this.manager) {
      this.manager.destroy();
    }
  }
}
```

## Conclusion

This migration guide provides a structured approach to converting the React-based Flash application to vanilla HTML/JavaScript while maintaining all functionality. The key is to preserve the existing business logic in the utility files while replacing React's component and state management patterns with vanilla JavaScript equivalents.

The migration can be done incrementally, testing each component as it's converted. Since the application already uses modern Web APIs and has a clean separation between UI and business logic, the migration should be straightforward.