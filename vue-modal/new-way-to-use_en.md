In this article I will raise the topic of using modal windows for web applications. I won't touch `<dialog/>` from HTML. We will talk about modal windows in the `vue` ecosystem. Codes and the implementation can be viewed here [`jenesius-vue-modal`](https://github.com/Jenesius/vue-modal).

The new `teleport` feature in the third version of `vue` allows you to move content to the location you need. And that's cool! Maybe. having tried to use it on my project, my team encountered the following problems: how to support such code when any component can insert something anywhere in your product.

We added abstractions, new components that were only used to limit the use of `teleport`.

We used a lot of modal windows in our project. Open one, open another on top, return value from modal, integrate modal window in `vue-router` is not a complete list of what we implemented.

At first we used a cool library, but like this the approach irritated our entire team:

```html
<!--widget-user-item.vue-->
<div>
    <div><!--The main content--></div>
    <teleport>
        <modal-user :user-id = "id"  v-if = "isActive"/>
    </teleport>
</div>
```

For us, It looked awkward and we really wanted to get rid of it. We wanted to get rid of the description of the modal components windows in another component. Our task was to make modal windows to another level, another layer.

In `vue` it is very convenient to use reactive states based on `ref`/`reactive`. Add a queue that will store open modal windows:

```ts
const modalQueue = reactive([]);
```

When adding a modal window, we will write the value there, when removing - delete. Everything seems to be very simple. Now to open a modal window add the `openModal` method:

```javascript
function openModal(component, props) {
    modalQueue.push({component, props});
}
```

To close we use the `splice` method.

Now all that remains is to display the modal windows. Let's create a separate one for this component in which the transferred components will be displayed. Also don't forget to pass `props` there:


```html
<!--modal-container.vue-->
<div>
    <component 
        v-for = "item in modalQueue"
        :is = "item.component" 
        v-bind = "item.props"
    />        
</div>
```


To open a modal window from our new user card, let's add a click handler and call a new function:

```vue
<template>
    <!--widget-user-item.vue-->
    <button @click = "showUserModal">open</button>
</template>
<script setup>
    const props = defineProps(['userId'])
    
    function showUserModal() {
        openModal(ModalUser, {
            userId: props.modalId
        })
    }
</script>
```
also, we must connect this component to `App.vue`.

This is just a short but main part of what we have implemented. For us this approach allows you to build interfaces more accurately and safely.

The entire code base can be found in the public repository at [GitHub](https://github.com/Jenesius/vue-modal). We tried to give
examples on how to use this library. We also described instructions and made nice documentation.