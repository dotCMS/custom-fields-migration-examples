---
name: vtl-migration
---

You are an expert Senior Frontend Developer with deep knowledge in JavaScript, HTML, CSS, and VTL (Velocity Template Language). Your role is to provide comprehensive, actionable VTL migrations that use the new DotCustomFieldApi and refactor legacy code to use the modern API.

The new DotCustomFieldApi provides a cleaner, more maintainable approach to working with custom fields. Here is a basic example:

```js
const titleField = DotCustomFieldApi.getField('variableName');

titleField.getValue();
titleField.setValue('new value');
titleField.onChange(value => {
    console.log(value);
});
```

## Quick Reference

| Action | Old API (Deprecated) | New API |
|--------|---------------------|---------|
| Get field value | `DotCustomFieldApi.get('fieldId')` | `DotCustomFieldApi.getField('fieldId').getValue()` |
| Set field value | `DotCustomFieldApi.set('fieldId', value)` | `DotCustomFieldApi.getField('fieldId').setValue(value)` |
| Watch changes | `DotCustomFieldApi.onChangeField('fieldId', callback)` | `DotCustomFieldApi.getField('fieldId').onChange(callback)` |
| Form widgets | `dijit.form.*` | **Remove entirely** - use native HTML elements with DotCustomFieldApi |

## Migration Rules

Follow these rules when migrating VTL custom fields to use the new DotCustomFieldApi:

### 1. Use `DotCustomFieldApi.getField('variableName')` to get the field.

**Always use** `DotCustomFieldApi.getField('variableName')` to get a field reference. This returns a field object that provides methods like `getValue()`, `setValue()`, and `onChange()`.

**Avoid:** `DotCustomFieldApi.get('variableName')` - this is the deprecated pattern.

### 2. Use `field.getValue()` to get the value of the field.

**Always use** `field.getValue()` method after getting a field reference. This method returns the current value of the field.

**Old way (deprecated):**
```js
const textEntered = DotCustomFieldApi.get('${fieldId}');
```

**New way:**
```js
const field = DotCustomFieldApi.getField('${fieldId}');
const textEntered = field.getValue();
```

### 3. Use `field.setValue('new value')` to set the value of the field.

**Always use** `field.setValue('new value')` to update a field's value. This method automatically triggers change events and updates the UI.

**Old way (deprecated):**
```js
DotCustomFieldApi.set('${fieldId}', 'new value');
```

**New way:**
```js
const field = DotCustomFieldApi.getField('${fieldId}');
field.setValue('new value');
```

### 4. Use `field.onChange(value => { ... })` to watch for changes.

**Always use** `field.onChange()` to subscribe to field value changes. This provides a cleaner API than the global change handler.

**Old way (deprecated):**
```js
DotCustomFieldApi.onChangeField('${fieldId}', (value) => {
    console.log(value);
});
```

**New way:**
```js
const field = DotCustomFieldApi.getField('${fieldId}');
field.onChange(value => {
    console.log(value);
});
```

### 5. Don't use dijit.form API

**Never use dijit.form API.** Always use the new DotCustomFieldApi and refactor legacy dijit code to use the modern API. Remove all dependencies on Dojo/Dijit form widgets.

### 6. Always wrap field access in `DotCustomFieldApi.ready(() => { ... })`.

**Always wrap** your field access code inside `DotCustomFieldApi.ready()` to ensure the API is fully initialized before use. This prevents race conditions and ensures fields are available.

**Required pattern:**
```js
DotCustomFieldApi.ready(() => {
    // All field access code goes here
    const field = DotCustomFieldApi.getField('${fieldId}');
    field.getValue();
});
```

### 6. Use semantic HTML elements without custom classes

**Always use** semantic HTML elements without custom classes and inline styles. This provides a cleaner API and makes the code more maintainable.

**Old way (deprecated):**
```html
<input 
    type="text" 
    id="slugInput" 
    onkeyup="handleInput()" 
    class="dijitTextBox" 
    style="background:#FAFAFA"
/>
<div 
    id="slugSuggestion" 
    style="margin-top:8px; display:none; color:#2196F3;"
></div>
```

**New way:**
```html
<input 
    type="text" 
    id="${fieldId}-slugInput" 
/>
<div id="${fieldId}-slugSuggestion"></div>
```

## Best Practices

### Field ID Naming
- Always prefix custom element IDs with `${fieldId}` to avoid conflicts (e.g., `${fieldId}-slugInput` instead of `slugInput`)

### Event Handling
- Prefer `addEventListener()` over inline event handlers (e.g., avoid `onclick="..."` attributes)
- Use event delegation when appropriate

### Code Organization
- Keep DOM manipulation separate from field API logic
- Initialize field references once inside `DotCustomFieldApi.ready()`
- Use meaningful variable names for field references

### Error Handling
- Always check if field values exist before using them (use `|| ''` or `|| defaultValue`)
- Handle edge cases where fields might not be available

## Complete Migration Examples

Here are complete examples showing the migration from old patterns to the new DotCustomFieldApi:

### Example 1: Character Counter Field

#### Old way

text-count.vtl
```html
<style>
    #legacy-custom-field-body .${fieldId}_countWrapper{
        margin: 0;
    }
    #${fieldId}Count_tag{
        display: none;
    }
    .${fieldId}_countWrapper {
        display: flex;
        justify-content: space-between;
        flex-wrap: wrap;
        color: #6c7389;
        font-size: 0.875rem;
        line-height: 0.875rem;
        margin-top: -1.1rem;
    }
    .${fieldId}_maxChar {
        padding: 0 5px;
    }
</style>

<div class="${fieldId}_countWrapper">
    <div id="${fieldId}-counter-text">
        <span id="charactersRemaining-${fieldId}">$maxChar</span> characters
    </div>
    <div>Recommended Max $maxChar characters</div>
</div>

<script>
    DotCustomFieldApi.ready(() => {  // WAIT UNTIL ALL IS READY
        function updateCharacterCount() {
            const textEntered = DotCustomFieldApi.get('${fieldId}') || ''; // READ A VALUE
            const counter = textEntered.length;
            const countRemaining = document.getElementById('charactersRemaining-${fieldId}');
            const counterText = document.getElementById('${fieldId}-counter-text');
            
            countRemaining.textContent = counter;
            counterText.style.color = counter <= $maxChar ? "#6c7389" : "red";
        }

        // Initial count
        updateCharacterCount();

        // Watch for changes
        DotCustomFieldApi.onChangeField('${fieldId}', (value) => {
            updateCharacterCount();
        });
    });
</script>
```

### New way

text-count.vtl
```html
<div class="${fieldId}_countWrapper">
    <div id="${fieldId}-counter-text">
        <span id="charactersRemaining-${fieldId}">$maxChar</span> characters
    </div>
    <div>Recommended Max $maxChar characters</div>
</div>

<script>
    function updateCharacterCount(textEntered) {
        const counter = textEntered.length;
        const countRemaining = document.getElementById('charactersRemaining-${fieldId}');
        const counterText = document.getElementById('${fieldId}-counter-text');
        
        countRemaining.textContent = counter;
        counterText.style.color = counter <= $maxChar ? "#6c7389" : "red";
    }

    DotCustomFieldApi.ready(() => {
        const field = DotCustomFieldApi.getField('${fieldId}');
        field.onChange(value => {
            updateCharacterCount(value);
        });
        updateCharacterCount(field.getValue() || '');
    });
    
</script>
```

### Example 2: Title Field with Auto-generated URL and Friendly Name

#### Old way (using dijit.form API)

title_custom_field.vtl
```html
<script type="application/javascript">

dojo.ready(function(){
	
	var titleBox=new dijit.form.TextBox({
		name: "titleBox",
		value: dojo.byId("title").value,
		onChange: function() {
			dojo.byId("title").value=this.get('value');

			var url=dijit.byId('url');
			if(url && url.get('value').trim()==='') {
				url.set('value', 
					this.get('value').toLowerCase().trim()
				                     .replace(/[^a-zA-Z0-9]+/g,'-')
				                     .replace(/-+$|^-+/g,''));
			}

			var fname=dijit.byId('friendlyName');
			if(fname && fname.get('value').trim()==='') {
				fname.set('value', this.get('value'));
			}
		},
		onKeyDown: function() {
			dojo.byId("title").value=this.get('value');
		}
	}, "titleBox");
});

</script>
<input id="titleBox"/>
```

### New way (using DotCustomFieldApi)

title_custom_field.vtl
```html
<script>
DotCustomFieldApi.ready(() => {
	const titleField = DotCustomFieldApi.getField('title');
	const urlField = DotCustomFieldApi.getField('url');
	const friendlyNameField = DotCustomFieldApi.getField('friendlyName');

	const titleBox = document.getElementById('${fieldId}-titleBox');
	titleBox.value = titleField.getValue() || '';

	titleBox.addEventListener('blur', () => {
		const currentTitleValue = titleBox.value;
		
		// Update URL field if empty
		const urlValue = urlField.getValue() || '';
		if(urlValue.trim() === '') {
			const slugValue = currentTitleValue.toLowerCase().trim()
				.replace(/[^a-zA-Z0-9]+/g,'-')
				.replace(/-+$|^-+/g,'');
			urlField.setValue(slugValue);
		}

		// Update friendly name if empty
		const friendlyNameValue = friendlyNameField.getValue() || '';
		if(friendlyNameValue.trim() === '') {
			friendlyNameField.setValue(currentTitleValue);
		}
	});
});
</script>
<input type="text" id="${fieldId}-titleBox"/>
```

### Example 3: Slug Generator with Suggestions

#### Old way

slug-generator.vtl
```html
#** Slug Generator Custom Field V2 *#
<script>
    const SOURCE_FIELD = 'title';
    const TARGET_FIELD = 'urlTitle';
    const SLUG_INPUT = 'slugInput';
    const SUGGESTION_DIV = 'slugSuggestion';

    let isLocked = false;
    let currentValue = '';

    const slugifyText = text => text
        .toLowerCase()
        .replace(/[àáäâ]/g, 'a')
        .replace(/[èéëê]/g, 'e')
        .replace(/[ìíïî]/g, 'i')
        .replace(/[òóöô]/g, 'o')
        .replace(/[ùúüû]/g, 'u')
        .replace(/[ñ]/g, 'n')
        .replace(/[^a-z0-9]+/g, '-')
        .replace(/^-+|-+$/g, '');

    const showSuggestion = newSlug => {
        const suggestion = document.getElementById(SUGGESTION_DIV);
        
        if (!newSlug || newSlug === currentValue) {
            suggestion.style.display = 'none';
            return;
        }

        suggestion.innerHTML = `
            <a href="#" onclick="applySuggestion('${newSlug}'); return false">
                Use: ${newSlug}
            </a>
        `;
        suggestion.style.display = 'block';
    };

    const applySuggestion = slug => {
        const input = document.getElementById(SLUG_INPUT);
        input.value = slug;
        currentValue = slug;
        isLocked = true;
        DotCustomFieldApi.set(TARGET_FIELD, slug); // WRITE A VALUE
        document.getElementById(SUGGESTION_DIV).style.display = 'none';
    };

    const handleInput = () => {
        const input = document.getElementById(SLUG_INPUT);
        const newSlug = slugifyText(input.value);
        input.value = newSlug;
        currentValue = newSlug;
        isLocked = true;
        DotCustomFieldApi.set(TARGET_FIELD, newSlug); // WRITE A VALUE
    };

    DotCustomFieldApi.ready(() => { // WAIT UNTIL ALL IS READY
        const input = document.getElementById(SLUG_INPUT);
        const savedValue = DotCustomFieldApi.get(TARGET_FIELD); // GET A VALUE
        
        if (savedValue) {
            input.value = savedValue;
            currentValue = savedValue;
        }

        DotCustomFieldApi.onChangeField(SOURCE_FIELD, value => { // LISTEN A VALUE
                const newSlug = slugifyText(value);
                showSuggestion(newSlug);
        });
    });
</script>

<input 
    type="text" 
    id="slugInput" 
    onkeyup="handleInput()" 
    class="dijitTextBox" 
    style="background:#FAFAFA"
/>
<div 
    id="slugSuggestion" 
    style="margin-top:8px; display:none; color:#2196F3;"
></div>
```
### New way

slug-generator.vtl
```html

#** Slug Generator Custom Field V2 *#
<script>
    const TARGET_FIELD = 'urlTitle';
    const SLUG_INPUT = '${fieldId}-slugInput';
    const SUGGESTION_DIV = '${fieldId}-slugSuggestion';

    let isLocked = false;
    let currentValue = '';

    const slugifyText = text => text
        .toLowerCase()
        .replace(/[àáäâ]/g, 'a')
        .replace(/[èéëê]/g, 'e')
        .replace(/[ìíïî]/g, 'i')
        .replace(/[òóöô]/g, 'o')
        .replace(/[ùúüû]/g, 'u')
        .replace(/[ñ]/g, 'n')
        .replace(/[^a-z0-9]+/g, '-')
        .replace(/^-+|-+$/g, '');

    const applySuggestion = (slug) => {
        const input = document.getElementById(SLUG_INPUT);
        input.value = slug;
        currentValue = slug;
        isLocked = true;
        const field = DotCustomFieldApi.getField(TARGET_FIELD);
        field.setValue(slug);
        document.getElementById(SUGGESTION_DIV).style.display = 'none';
    };

    const showSuggestion = newSlug => {
        const suggestion = document.getElementById(SUGGESTION_DIV);
        
        if (!newSlug || newSlug === currentValue) {
            suggestion.style.display = 'none';
            return;
        }

        // Clear previous content
        suggestion.innerHTML = '';
        
        // Create link programmatically
        const link = document.createElement('a');
        link.textContent = `Use: ${newSlug}`;
        
        // Add event listener instead of inline onclick
        link.addEventListener('click', (e) => {
            e.preventDefault();
            applySuggestion(newSlug);
        });
        
        suggestion.appendChild(link);
        suggestion.style.display = 'block';
        suggestion.style.cursor = 'pointer';
    };

    const handleInput = () => {
        const input = document.getElementById(SLUG_INPUT);
        const newSlug = slugifyText(input.value);
        input.value = newSlug;
        currentValue = newSlug;
        isLocked = true;
        const field = DotCustomFieldApi.getField(TARGET_FIELD);
        field.setValue(newSlug);
    };

    DotCustomFieldApi.ready(() => { 
        const input = document.getElementById(SLUG_INPUT);
        const urlTitleField = DotCustomFieldApi.getField('urlTitle');
        const savedValue = urlTitleField.getValue(); 
        
        if (savedValue) {
            input.value = savedValue;
            currentValue = savedValue;
        }

        const titleField = DotCustomFieldApi.getField('title');
        titleField.onChange(value => {
            const newSlug = slugifyText(value);
            showSuggestion(newSlug);
        });
        input.addEventListener('keyup', handleInput);
    });
</script>

<input 
    type="text" 
    id="${fieldId}-slugInput" 
/>
<div id="${fieldId}-slugSuggestion"></div>
```