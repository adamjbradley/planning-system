# 🎨 UI Component Pull Request

## Component Information
**Component Name:** 
**Component Type:** 
- [ ] New component
- [ ] Component update
- [ ] Design system addition
- [ ] Component library migration

## Design System Compliance
- [ ] Follows FinPath Explorer design system
- [ ] Uses ShadCN UI v2.3.0 base components
- [ ] Consistent with existing patterns
- [ ] Proper spacing/typography scale used

## Responsive Design
### Desktop (1920px+)
- [ ] Layout verified
- [ ] All functionality accessible
- [ ] Performance tested

### Tablet (768px - 1919px)
- [ ] Layout adapts properly
- [ ] Touch targets appropriate size
- [ ] Navigation remains intuitive

### Mobile (320px - 767px)
- [ ] Mobile-first design implemented
- [ ] Touch-friendly interactions
- [ ] Content hierarchy maintained
- [ ] Performance optimized

## Progressive Complexity Support
- [ ] **ELI5 Mode**: Simple, beginner-friendly interface
- [ ] **Intermediate Mode**: Additional options revealed gracefully
- [ ] **Expert Mode**: Full functionality and advanced controls
- [ ] Smooth transitions between complexity levels

## Accessibility (WCAG 2.1 AA)
- [ ] Semantic HTML structure
- [ ] Proper ARIA labels and roles
- [ ] Keyboard navigation support
- [ ] Screen reader compatibility
- [ ] Color contrast meets standards (4.5:1 minimum)
- [ ] Focus indicators visible
- [ ] Alternative text for images

## Theme Support
- [ ] Light theme implementation
- [ ] Dark theme implementation
- [ ] Theme switching tested
- [ ] CSS custom properties used
- [ ] No hardcoded colors

## Animation & Performance
- [ ] Animations run at 60fps
- [ ] Hardware acceleration used where appropriate
- [ ] No jank or stuttering
- [ ] Reduced motion preference respected
- [ ] Animation duration appropriate (200-500ms for most)

## Props & API
```typescript
interface ComponentProps {
  // Document the component API
}
```

## Usage Examples
```tsx
// Basic usage example

// Advanced usage example

// With different props
```

## Testing
- [ ] Unit tests written for component logic
- [ ] Interaction tests for user actions
- [ ] Visual regression tests added
- [ ] Accessibility tests included
- [ ] Performance tests if applicable

## Storybook Documentation
- [ ] Story created for each variant
- [ ] Controls/knobs configured
- [ ] Documentation page written
- [ ] Examples include best practices
- [ ] Accessibility notes included

## Screenshots/Videos
### Desktop
<!-- Add screenshots of desktop view -->

### Mobile
<!-- Add screenshots of mobile view -->

### Animations
<!-- Add video/GIF of animations if applicable -->

## Financial UI Considerations
- [ ] Currency formatting consistent
- [ ] Number precision appropriate for financial data
- [ ] Error states for invalid financial inputs
- [ ] Loading states for calculation delays
- [ ] Handles large financial numbers gracefully

## Browser Compatibility
- [ ] Chrome (latest 2 versions)
- [ ] Firefox (latest 2 versions)
- [ ] Safari (latest 2 versions)
- [ ] Edge (latest 2 versions)
- [ ] Mobile Safari (iOS 14+)
- [ ] Chrome Mobile (Android 10+)

## Code Quality
- [ ] TypeScript types complete and accurate
- [ ] Component follows React best practices
- [ ] No console errors or warnings
- [ ] Performance optimized (memo, callbacks, etc.)
- [ ] Clean, readable code with comments where needed

## Design Review
- [ ] UX team approved design
- [ ] Visual design matches Figma/mockups
- [ ] Interaction patterns consistent
- [ ] User flow logical and intuitive