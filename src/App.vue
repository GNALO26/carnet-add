<script setup>
import { ref, reactive, computed, watch, onMounted } from 'vue'

// --- État ---
const contacts = ref([])
const searchQuery = ref('')
const editingId = ref(null)

const form = reactive({
  nom: '',
  prenom: '',
  email: '',
  telephone: '',
  ville: '',
  codePostal: ''
})

// --- Chargement initial ---
onMounted(() => {
  const saved = localStorage.getItem('address-book')
  if (saved) {
    try {
      contacts.value = JSON.parse(saved)
    } catch (e) {
      contacts.value = []
    }
  }
  // Petite donnée de démo si vide
  if (contacts.value.length === 0) {
    contacts.value = [
      { id: 1, nom: 'Dupuis', prenom: 'Marie', email: 'marie@example.com', telephone: '0612345678', ville: 'Paris', codePostal: '75001' },
      { id: 2, nom: 'Lemoine', prenom: 'Paul', email: 'paul@example.com', telephone: '0698765432', ville: 'Lyon', codePostal: '69001' }
    ]
    // Pour éviter les conflits d'ID, on initialise nextId
  }
})

// Pour les nouveaux IDs
let nextId = contacts.value.length > 0 ? Math.max(...contacts.value.map(c => c.id)) + 1 : 1

// --- Sauvegarde automatique ---
watch(contacts, (newVal) => {
  localStorage.setItem('address-book', JSON.stringify(newVal))
}, { deep: true })

// --- Computed ---
const filteredContacts = computed(() => {
  const q = searchQuery.value.toLowerCase().trim()
  if (!q) return contacts.value
  return contacts.value.filter(c =>
    c.nom.toLowerCase().includes(q) ||
    c.prenom.toLowerCase().includes(q) ||
    c.email.toLowerCase().includes(q) ||
    c.ville.toLowerCase().includes(q)
  )
})

const stats = computed(() => {
  const total = contacts.value.length
  const cities = {}
  contacts.value.forEach(c => {
    cities[c.ville] = (cities[c.ville] || 0) + 1
  })
  const cityList = Object.entries(cities).map(([ville, count]) => ({ ville, count }))
  return { total, cityList }
})

// --- Méthodes ---
function resetForm() {
  form.nom = ''
  form.prenom = ''
  form.email = ''
  form.telephone = ''
  form.ville = ''
  form.codePostal = ''
  editingId.value = null
}

function editContact(contact) {
  form.nom = contact.nom
  form.prenom = contact.prenom
  form.email = contact.email
  form.telephone = contact.telephone
  form.ville = contact.ville
  form.codePostal = contact.codePostal
  editingId.value = contact.id
}

function saveContact() {
  // Validation simple
  if (!form.nom.trim() || !form.prenom.trim() || !form.email.trim()) {
    alert('Les champs Nom, Prénom et Email sont requis.')
    return
  }
  const contact = {
    id: editingId.value || nextId++,
    nom: form.nom.trim(),
    prenom: form.prenom.trim(),
    email: form.email.trim(),
    telephone: form.telephone.trim(),
    ville: form.ville.trim(),
    codePostal: form.codePostal.trim()
  }
  if (editingId.value) {
    const index = contacts.value.findIndex(c => c.id === editingId.value)
    if (index !== -1) contacts.value[index] = contact
  } else {
    contacts.value.push(contact)
  }
  resetForm()
}

function deleteContact(id) {
  contacts.value = contacts.value.filter(c => c.id !== id)
  if (editingId.value === id) resetForm()
}
</script>

<template>
  <div class="app-layout">
    <!-- Barre latérale gauche : statistiques -->
    <aside class="sidebar">
      <h3>Carnet d'adresses</h3>
      <div class="stat-card">
        <span class="stat-label">Total contacts</span>
        <span class="stat-value">{{ stats.total }}</span>
      </div>
      <div class="stat-card">
        <span class="stat-label">Par ville</span>
        <ul v-if="stats.cityList.length">
          <li v-for="city in stats.cityList" :key="city.ville">
            {{ city.ville }} : {{ city.count }}
          </li>
        </ul>
        <p v-else class="text-muted">Aucune donnée</p>
      </div>
    </aside>

    <!-- Contenu principal -->
    <main class="main">
      <h2>Gestion des contacts</h2>

      <!-- Formulaire -->
      <form @submit.prevent="saveContact" class="contact-form">
        <div class="form-row">
          <div class="form-group">
            <label>Nom</label>
            <input v-model="form.nom" type="text" placeholder="Nom" required />
          </div>
          <div class="form-group">
            <label>Prénom</label>
            <input v-model="form.prenom" type="text" placeholder="Prénom" required />
          </div>
        </div>
        <div class="form-row">
          <div class="form-group">
            <label>Email</label>
            <input v-model="form.email" type="email" placeholder="email@exemple.com" required />
          </div>
          <div class="form-group">
            <label>Téléphone</label>
            <input v-model="form.telephone" type="tel" placeholder="0612345678" />
          </div>
        </div>
        <div class="form-row">
          <div class="form-group">
            <label>Ville</label>
            <input v-model="form.ville" type="text" placeholder="Ville" />
          </div>
          <div class="form-group">
            <label>Code Postal</label>
            <input v-model="form.codePostal" type="text" placeholder="75001" />
          </div>
        </div>
        <div class="form-actions">
          <button type="submit" class="btn-primary">{{ editingId ? 'Modifier' : 'Ajouter' }}</button>
          <button v-if="editingId" type="button" @click="resetForm" class="btn-secondary">Annuler</button>
        </div>
      </form>

      <!-- Recherche -->
      <div class="search-bar">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Rechercher un contact..."
        />
      </div>

      <!-- Liste des contacts -->
      <div class="contact-list">
        <div v-if="filteredContacts.length === 0" class="empty">
          <p>Aucun contact trouvé.</p>
        </div>
        <table v-else class="table">
          <thead>
            <tr>
              <th>Nom</th>
              <th>Prénom</th>
              <th>Email</th>
              <th>Téléphone</th>
              <th>Ville</th>
              <th>Code Postal</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="contact in filteredContacts" :key="contact.id">
              <td>{{ contact.nom }}</td>
              <td>{{ contact.prenom }}</td>
              <td>{{ contact.email }}</td>
              <td>{{ contact.telephone }}</td>
              <td>{{ contact.ville }}</td>
              <td>{{ contact.codePostal }}</td>
              <td class="actions-cell">
                <button @click="editContact(contact)" class="btn-icon" title="Modifier">✎</button>
                <button @click="deleteContact(contact.id)" class="btn-icon delete" title="Supprimer">✕</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </main>
  </div>
</template>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.app-layout {
  display: flex;
  min-height: 100vh;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: #f5f5f7;
  color: #1c1c1e;
}

/* SIDEBAR */
.sidebar {
  width: 240px;
  background: #ffffff;
  padding: 24px;
  border-right: 1px solid #e5e5ea;
  box-shadow: 2px 0 8px rgba(0,0,0,0.02);
}

.sidebar h3 {
  font-size: 1.1rem;
  font-weight: 600;
  margin-bottom: 24px;
  color: #1c1c1e;
}

.stat-card {
  background: #f9f9fb;
  border-radius: 12px;
  padding: 14px;
  margin-bottom: 16px;
}

.stat-label {
  display: block;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #8e8e93;
  margin-bottom: 6px;
}

.stat-value {
  font-size: 1.4rem;
  font-weight: 600;
}

.stat-card ul {
  list-style: none;
  padding: 0;
  margin-top: 8px;
}

.stat-card li {
  padding: 3px 0;
  font-size: 0.85rem;
  color: #3a3a3c;
}

.text-muted {
  color: #8e8e93;
  font-size: 0.85rem;
}

/* MAIN */
.main {
  flex: 1;
  padding: 32px;
  overflow-y: auto;
}

.main h2 {
  font-size: 1.6rem;
  font-weight: 600;
  margin-bottom: 24px;
}

/* FORM */
.contact-form {
  background: white;
  border-radius: 16px;
  padding: 24px;
  margin-bottom: 32px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.04);
}

.form-row {
  display: flex;
  gap: 16px;
  margin-bottom: 16px;
}

.form-group {
  flex: 1;
}

.form-group label {
  display: block;
  font-size: 0.85rem;
  font-weight: 500;
  margin-bottom: 4px;
  color: #3a3a3c;
}

.form-group input {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #d1d1d6;
  border-radius: 10px;
  font-size: 0.95rem;
  background: #fafafa;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.form-group input:focus {
  border-color: #007aff;
  box-shadow: 0 0 0 3px rgba(0,122,255,0.1);
  background: #fff;
}

.form-actions {
  display: flex;
  gap: 10px;
  margin-top: 8px;
}

.btn-primary {
  padding: 10px 20px;
  background: #007aff;
  color: white;
  border: none;
  border-radius: 10px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s;
}

.btn-primary:hover {
  background: #0066d6;
}

.btn-secondary {
  padding: 10px 20px;
  background: #e5e5ea;
  color: #1c1c1e;
  border: none;
  border-radius: 10px;
  font-weight: 500;
  cursor: pointer;
}

.btn-secondary:hover {
  background: #d1d1d6;
}

/* SEARCH */
.search-bar {
  margin-bottom: 20px;
}

.search-bar input {
  width: 100%;
  padding: 10px 14px;
  border: 1px solid #d1d1d6;
  border-radius: 12px;
  font-size: 0.95rem;
  background: white;
  transition: border-color 0.2s;
}

.search-bar input:focus {
  border-color: #007aff;
}

/* TABLE */
.contact-list {
  background: white;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.04);
}

.table {
  width: 100%;
  border-collapse: collapse;
}

.table th,
.table td {
  text-align: left;
  padding: 10px 8px;
  border-bottom: 1px solid #f0f0f5;
}

.table th {
  font-size: 0.75rem;
  text-transform: uppercase;
  color: #8e8e93;
  font-weight: 600;
}

.table tbody tr:hover {
  background: #fafafa;
}

.actions-cell {
  display: flex;
  gap: 6px;
}

.btn-icon {
  background: none;
  border: 1px solid #d1d1d6;
  border-radius: 8px;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 0.9rem;
  color: #3a3a3c;
  transition: all 0.2s;
}

.btn-icon:hover {
  background: #f0f0f5;
  border-color: #007aff;
  color: #007aff;
}

.btn-icon.delete:hover {
  background: #ffeeed;
  border-color: #ff3b30;
  color: #ff3b30;
}

.empty {
  text-align: center;
  color: #8e8e93;
  padding: 24px;
}
</style>