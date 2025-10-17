## 1. Vue 3 (Inertia.js) Upload Form
#### File: `ProfileForm.vue`

```js
<template>
  <form @submit.prevent="updateProfile">
    <div>
      <label>Name</label>
      <input type="text" v-model="form.name" />
      <div v-if="form.errors.name">{{ form.errors.name }}</div>
    </div>
    <div>
      <label>Email</label>
      <input type="email" v-model="form.email" />
      <div v-if="form.errors.email">{{ form.errors.email }}</div>
    </div>
    <div>
      <label>Avatar</label>
      <img v-if="avatarPreview" :src="avatarPreview" alt="Avatar" style="max-width: 100px;"/>
      <input type="file" @change="handleAvatarChange" accept="image/*" />
      <div v-if="form.errors.avatar">{{ form.errors.avatar }}</div>
    </div>
    <progress v-if="form.progress" :value="form.progress.percentage" max="100"/>
    <button type="submit" :disabled="form.processing">
      Update Profile
    </button>
  </form>
</template>

<script setup>
import { ref } from 'vue'
import { useForm } from '@inertiajs/vue3'

const props = defineProps({ user: Object })

const form = useForm({
  _method: 'put',
  name: props.user.name,
  email: props.user.email,
  avatar: null
})

const avatarPreview = ref(props.user.avatar_url)

function handleAvatarChange(event) {
  const file = event.target.files[0]
  if (file) {
    form.avatar = file
    avatarPreview.value = URL.createObjectURL(file)
  }
}

function updateProfile() {
  form.post(route('profile.update'), {
    preserveScroll: true,
    onSuccess: () => {
      // Optionally reset the form or handle success
    }
  })
}
</script>
```

## 2. React (Inertia.js) Upload Form
#### File: `ProfileForm.jsx`
```js
import React, { useState } from 'react'
import { useForm } from '@inertiajs/react'

export default function ProfileForm({ user }) {
  const [avatarPreview, setAvatarPreview] = useState(user.avatar_url)
  const { data, setData, post, processing, errors, progress } = useForm({
    _method: 'put',
    name: user.name,
    email: user.email,
    avatar: null
  })

  const handleAvatarChange = (e) => {
    const file = e.target.files[0]
    if (file) {
      setData('avatar', file)
      setAvatarPreview(URL.createObjectURL(file))
    }
  }

  const handleSubmit = (e) => {
    e.preventDefault()
    post(route('profile.update'))
  }

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Name</label>
        <input
          type="text"
          value={data.name}
          onChange={e => setData('name', e.target.value)}
        />
        {errors.name && <div>{errors.name}</div>}
      </div>
      <div>
        <label>Email</label>
        <input
          type="email"
          value={data.email}
          onChange={e => setData('email', e.target.value)}
        />
        {errors.email && <div>{errors.email}</div>}
      </div>
      <div>
        <label>Avatar</label>
        {avatarPreview && (
          <img src={avatarPreview} alt="Avatar" style={{ maxWidth: '100px' }} />
        )}
        <input type="file" accept="image/*" onChange={handleAvatarChange} />
        {errors.avatar && <div>{errors.avatar}</div>}
      </div>
      {progress && (
        <progress value={progress.percentage} max={100} />
      )}
      <button type="submit" disabled={processing}>
        Update Profile
      </button>
    </form>
  )
}
```

## 3. Laravel Livewire Upload Form
#### File: `ProfileForm.php` (Livewire Component Class)
```php
<?php

namespace App\Http\Livewire;

use Illuminate\Support\Facades\Storage;
use Livewire\Component;
use Livewire\WithFileUploads;

class ProfileForm extends Component
{
    use WithFileUploads;

    public $name;
    public $email;
    public $avatar;
    public $avatarPreview;

    public function mount($user)
    {
        $this->name = $user->name;
        $this->email = $user->email;
        $this->avatarPreview = $user->avatar_url;
    }

    public function updatedAvatar()
    {
        $this->avatarPreview = $this->avatar->temporaryUrl();
    }

    public function updateProfile()
    {
        $this->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users,email,' . auth()->id(),
            'avatar' => 'nullable|image|max:2048',
        ]);

        $user = auth()->user();

        if ($this->avatar) {
            if ($user->avatar_path) {
                Storage::delete($user->avatar_path);
            }
            $path = $this->avatar->store('avatars', 'public');
            $user->avatar_path = $path;
        }

        $user->name = $this->name;
        $user->email = $this->email;
        $user->save();

        session()->flash('success', 'Profile updated successfully.');
    }

    public function render()
    {
        return view('livewire.profile-form');
    }
}
```

File: `profile-form.blade.php` (Livewire Blade View)
```php
<form wire:submit.prevent="updateProfile">
  <div>
    <label>Name</label>
    <input type="text" wire:model="name">
    @error('name') <span>{{ $message }}</span> @enderror
  </div>
  <div>
    <label>Email</label>
    <input type="email" wire:model="email">
    @error('email') <span>{{ $message }}</span> @enderror
  </div>
  <div>
    <label>Avatar</label>
    @if($avatarPreview)
      <img src="{{ $avatarPreview }}" alt="Avatar" style="max-width: 100px;">
    @endif
    <input type="file" wire:model="avatar" accept="image/*">
    @error('avatar') <span>{{ $message }}</span> @enderror
  </div>
  <button type="submit">Update Profile</button>
  @if(session('success'))
    <div>{{ session('success') }}</div>
  @endif
</form>
```