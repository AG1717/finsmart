# 🔔 Guide du Système de Notifications Admin - FinSmart

## ✅ Ce qui a été implémenté

### Backend Complet ✅

#### 1. Modèle AdminNotification
**Fichier**: `finsmart-backend/src/models/AdminNotification.model.js`

Types de notifications:
- **user_registered** - Nouvel utilisateur inscrit
- **user_first_goal** - Premier objectif créé
- **user_milestone** - Milestone atteint (5/10 objectifs, etc.)
- **goal_completed** - Objectif complété
- **goal_high_value** - Objectif de grande valeur (>$10,000)
- **admin_action** - Action admin (modification, suppression)
- **suspicious_activity** - Activité suspecte

Niveaux de sévérité:
- `info` - Information
- `success` - Succès
- `warning` - Avertissement
- `critical` - Critique

#### 2. Service de Notifications
**Fichier**: `finsmart-backend/src/services/notification.service.js`

Fonctions disponibles:
```javascript
// Événements utilisateurs
NotificationService.notifyNewUser(user)
NotificationService.notifyFirstGoal(user, goal)
NotificationService.notifyGoalCompleted(user, goal)
NotificationService.notifyHighValueGoal(user, goal)
NotificationService.notifyUserMilestone(user, milestoneType, data)

// Actions admin
NotificationService.logAdminAction(admin, action, target, details)

// Activité suspecte
NotificationService.notifySuspiciousActivity(type, data)

// Gestion
NotificationService.getNotifications(filters, options)
NotificationService.markAllAsRead()
NotificationService.getUnreadCount()
NotificationService.cleanOldNotifications(days)
```

#### 3. Routes API
**Endpoint de base**: `/api/v1/admin/notifications`

```bash
GET    /api/v1/admin/notifications              # Liste avec filtres
GET    /api/v1/admin/notifications/stats        # Statistiques
GET    /api/v1/admin/notifications/unread-count # Nombre non lues
PUT    /api/v1/admin/notifications/:id/read     # Marquer comme lue
PUT    /api/v1/admin/notifications/mark-all-read # Tout marquer comme lu
DELETE /api/v1/admin/notifications/:id          # Supprimer une notification
DELETE /api/v1/admin/notifications/cleanup      # Nettoyer anciennes
```

#### 4. Triggers Automatiques

Les notifications sont créées automatiquement lors:

**Inscription d'un utilisateur**:
- `auth.controller.js` → Appelle `notifyNewUser()`
- Notification: "🎉 Nouvel utilisateur inscrit"

**Création d'objectif**:
- `goal.controller.js` → V érifications multiples
  - Premier objectif → `notifyFirstGoal()`
  - Grande valeur → `notifyHighValueGoal()`
  - Milestones 5/10 objectifs → `notifyUserMilestone()`

**Complétion d'objectif**:
- `goal.controller.js` → `notifyGoalCompleted()`
- Milestones: 1ère/5ème completion

**Actions admin**:
- Modification utilisateur → `logAdminAction('user_updated')`
- Suppression utilisateur → `logAdminAction('user_deleted')`
- Promotion admin → `logAdminAction('user_promoted')`
- Modification objectif → `logAdminAction('goal_updated')`
- Suppression objectif → `logAdminAction('goal_deleted')`

## 🚀 Comment utiliser

### 1. Le backend crée automatiquement les notifications

Aucune action nécessaire! Dès qu'un événement se produit, une notification est créée.

### 2. Tester les notifications

#### A. Créer des notifications de test

**Inscrivez un nouvel utilisateur** via l'app mobile:
```bash
# Une notification sera créée automatiquement
```

**Créez un objectif**:
```bash
# Notifications possibles:
# - Premier objectif de l'utilisateur
# - Objectif de grande valeur (si >$10,000)
# - Milestone (si 5ème ou 10ème objectif)
```

**Complétez un objectif**:
```bash
# Notifications:
# - Objectif complété
# - Première completion (si 1er)
# - 5ème completion (si 5ème)
```

**Effectuez une action admin**:
```bash
# Dans le dashboard admin:
# - Modifiez un utilisateur → Notification "⚙️ Action admin"
# - Supprimez un objectif → Notification avec sévérité "warning"
# - Promouvez en admin → Notification avec sévérité "warning"
```

#### B. Récupérer les notifications via API

```bash
# Toutes les notifications
curl -H "Authorization: Bearer <admin_token>" \
  http://localhost:3000/api/v1/admin/notifications

# Filtrer par type
curl -H "Authorization: Bearer <admin_token>" \
  "http://localhost:3000/api/v1/admin/notifications?type=user_registered"

# Filtrer par sévérité
curl -H "Authorization: Bearer <admin_token>" \
  "http://localhost:3000/api/v1/admin/notifications?severity=critical"

# Non lues uniquement
curl -H "Authorization: Bearer <admin_token>" \
  "http://localhost:3000/api/v1/admin/notifications?isRead=false"

# Statistiques
curl -H "Authorization: Bearer <admin_token>" \
  http://localhost:3000/api/v1/admin/notifications/stats

# Nombre non lues
curl -H "Authorization: Bearer <admin_token>" \
  http://localhost:3000/api/v1/admin/notifications/unread-count
```

### 3. Marquer comme lu

```bash
# Une notification spécifique
curl -X PUT -H "Authorization: Bearer <admin_token>" \
  http://localhost:3000/api/v1/admin/notifications/NOTIFICATION_ID/read

# Toutes les notifications
curl -X PUT -H "Authorization: Bearer <admin_token>" \
  http://localhost:3000/api/v1/admin/notifications/mark-all-read
```

### 4. Nettoyer les anciennes notifications

```bash
# Supprimer les notifications lues de plus de 30 jours (défaut)
curl -X DELETE -H "Authorization: Bearer <admin_token>" \
  http://localhost:3000/api/v1/admin/notifications/cleanup

# Personnaliser le nombre de jours
curl -X DELETE -H "Authorization: Bearer <admin_token>" \
  "http://localhost:3000/api/v1/admin/notifications/cleanup?days=60"
```

## 📊 Exemples de Notifications

### Nouvel utilisateur
```json
{
  "type": "user_registered",
  "title": "🎉 Nouvel utilisateur inscrit",
  "message": "john_doe (john@example.com) vient de s'inscrire",
  "severity": "info",
  "userId": "67890abcdef",
  "metadata": {
    "username": "john_doe",
    "email": "john@example.com",
    "currency": "USD"
  },
  "isRead": false,
  "createdAt": "2026-01-17T10:30:00Z"
}
```

### Premier objectif
```json
{
  "type": "user_first_goal",
  "title": "🎯 Premier objectif créé",
  "message": "john_doe a créé son premier objectif: \"Économiser pour vacances\"",
  "severity": "success",
  "userId": "67890abcdef",
  "goalId": "12345xyz",
  "metadata": {
    "username": "john_doe",
    "goalName": "Économiser pour vacances",
    "targetAmount": 5000,
    "currency": "$"
  }
}
```

### Action admin
```json
{
  "type": "admin_action",
  "title": "⚙️ Action admin",
  "message": "admin_user a promu john_doe en admin",
  "severity": "warning",
  "adminId": "admin123",
  "userId": "67890abcdef",
  "metadata": {
    "adminUsername": "admin_user",
    "action": "user_promoted",
    "roleChanged": true,
    "oldRole": "user",
    "newRole": "admin"
  }
}
```

### Objectif complété
```json
{
  "type": "goal_completed",
  "title": "✅ Objectif atteint!",
  "message": "john_doe a atteint son objectif \"Économiser pour vacances\"",
  "severity": "success",
  "userId": "67890abcdef",
  "goalId": "12345xyz",
  "metadata": {
    "username": "john_doe",
    "goalName": "Économiser pour vacances",
    "amount": 5000,
    "currency": "$",
    "category": "lifestyle",
    "timeframe": "short"
  }
}
```

## 🎨 Frontend (À implémenter)

### Structure recommandée

#### 1. Onglet Notifications dans le dashboard

Ajoutez dans `index.html`:
```html
<div class="nav-tabs">
    <button class="nav-tab active" data-tab="analytics">📊 Analytics</button>
    <button class="nav-tab" data-tab="users">👥 Users</button>
    <button class="nav-tab" data-tab="goals">🎯 Goals</button>
    <button class="nav-tab" data-tab="notifications">🔔 Notifications</button>
</div>

<!-- Notifications Tab -->
<div id="notificationsTab" class="tab-content">
    <!-- Badge non lues -->
    <div class="notifications-header">
        <h2>Notifications</h2>
        <span class="unread-badge" id="unreadCount">0</span>
        <button id="markAllReadBtn" class="btn-secondary">Mark all as read</button>
    </div>

    <!-- Filtres -->
    <div class="search-bar">
        <select id="notifTypeFilter">
            <option value="">All Types</option>
            <option value="user_registered">New Users</option>
            <option value="goal_completed">Goals Completed</option>
            <option value="admin_action">Admin Actions</option>
        </select>
        <select id="notifSeverityFilter">
            <option value="">All Severities</option>
            <option value="info">Info</option>
            <option value="success">Success</option>
            <option value="warning">Warning</option>
            <option value="critical">Critical</option>
        </select>
        <select id="notifReadFilter">
            <option value="">All</option>
            <option value="false">Unread Only</option>
            <option value="true">Read Only</option>
        </select>
    </div>

    <!-- Liste des notifications -->
    <div id="notificationsList"></div>
    <div id="notificationsPagination" class="pagination"></div>
</div>
```

#### 2. Styles CSS

```css
/* Notifications */
.notifications-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
}

.unread-badge {
    background: #D32F2F;
    color: white;
    padding: 4px 12px;
    border-radius: 12px;
    font-size: 14px;
    font-weight: 600;
}

.notification-item {
    background: white;
    padding: 16px;
    border-radius: 8px;
    margin-bottom: 12px;
    border-left: 4px solid #00C9A7;
    display: flex;
    justify-content: space-between;
    align-items: start;
}

.notification-item.unread {
    background: #F0F8FF;
}

.notification-item.severity-warning {
    border-left-color: #FFA500;
}

.notification-item.severity-critical {
    border-left-color: #D32F2F;
}

.notification-content {
    flex: 1;
}

.notification-title {
    font-weight: 600;
    margin-bottom: 4px;
}

.notification-message {
    color: #666;
    font-size: 14px;
}

.notification-time {
    color: #999;
    font-size: 12px;
    margin-top: 8px;
}
```

#### 3. JavaScript Logic

```javascript
// Fetch notifications
async function fetchNotifications(page = 1) {
    const type = document.getElementById('notifTypeFilter').value;
    const severity = document.getElementById('notifSeverityFilter').value;
    const isRead = document.getElementById('notifReadFilter').value;

    const params = new URLSearchParams({
        page,
        limit: 20,
        ...(type && { type }),
        ...(severity && { severity }),
        ...(isRead && { isRead })
    });

    const response = await fetch(`${API_URL}/admin/notifications?${params}`, {
        headers: { 'Authorization': `Bearer ${accessToken}` }
    });

    const data = await response.json();
    if (data.success) {
        displayNotifications(data.data.notifications, data.data.pagination);
        updateUnreadCount();
    }
}

// Display notifications
function displayNotifications(notifications, pagination) {
    const container = document.getElementById('notificationsList');
    container.innerHTML = '';

    notifications.forEach(notif => {
        const div = document.createElement('div');
        div.className = `notification-item ${!notif.isRead ? 'unread' : ''} severity-${notif.severity}`;
        div.innerHTML = `
            <div class="notification-content">
                <div class="notification-title">${notif.title}</div>
                <div class="notification-message">${notif.message}</div>
                <div class="notification-time">${formatTime(notif.createdAt)}</div>
            </div>
            <div class="notification-actions">
                ${!notif.isRead ? `<button onclick="markAsRead('${notif._id}')">Mark Read</button>` : ''}
                <button onclick="deleteNotification('${notif._id}')">×</button>
            </div>
        `;
        container.appendChild(div);
    });
}

// Update unread count
async function updateUnreadCount() {
    const response = await fetch(`${API_URL}/admin/notifications/unread-count`, {
        headers: { 'Authorization': `Bearer ${accessToken}` }
    });
    const data = await response.json();
    if (data.success) {
        document.getElementById('unreadCount').textContent = data.data.count;
    }
}

// Mark as read
async function markAsRead(id) {
    await fetch(`${API_URL}/admin/notifications/${id}/read`, {
        method: 'PUT',
        headers: { 'Authorization': `Bearer ${accessToken}` }
    });
    fetchNotifications();
}

// Mark all as read
document.getElementById('markAllReadBtn').addEventListener('click', async () => {
    await fetch(`${API_URL}/admin/notifications/mark-all-read`, {
        method: 'PUT',
        headers: { 'Authorization': `Bearer ${accessToken}` }
    });
    fetchNotifications();
});

// Delete notification
async function deleteNotification(id) {
    if (confirm('Delete this notification?')) {
        await fetch(`${API_URL}/admin/notifications/${id}`, {
            method: 'DELETE',
            headers: { 'Authorization': `Bearer ${accessToken}` }
        });
        fetchNotifications();
    }
}
```

## 🎯 Prochaines étapes

Pour compléter l'implémentation:

1. **Ajouter l'onglet Notifications** au dashboard HTML
2. **Ajouter les styles** CSS pour les notifications
3. **Implémenter la logique** JavaScript de récupération et affichage
4. **Tester** avec des données réelles
5. **(Optionnel)** Ajouter des notifications en temps réel avec WebSockets

## 🔍 Debugging

Pour voir toutes les notifications créées:
```bash
# Avec MongoDB shell
use finsmart
db.adminnotifications.find().sort({createdAt: -1}).limit(10)

# Compter par type
db.adminnotifications.aggregate([
  { $group: { _id: "$type", count: { $sum: 1 } } }
])
```

Pour vérifier si les triggers fonctionnent:
```bash
# Regardez les logs backend quand un événement se produit
# Vous devriez voir: "Admin notification created: ..."
```

## ✅ Résumé

Vous avez maintenant:
- ✅ Un système complet de notifications backend
- ✅ Notifications automatiques pour tous les événements importants
- ✅ Logs de toutes les actions admin
- ✅ API complète pour gérer les notifications
- ✅ Milestones et achievements trackés
- ⏳ Frontend à implémenter (structure fournie ci-dessus)

Le système est prêt à l'emploi! Il suffit d'ajouter l'interface dans le dashboard admin.
