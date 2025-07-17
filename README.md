<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MediaTracker</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap');
        
        :root {
            --primary: #6366f1;
            --primary-dark: #4f46e5;
            --secondary: #f43f5e;
            --dark: #1e293b;
            --light: #f8fafc;
            --glass: rgba(255, 255, 255, 0.15);
        }
        
        body {
            font-family: 'Poppins', sans-serif;
            background: linear-gradient(135deg, #0f172a, #1e293b);
            color: var(--light);
            min-height: 100vh;
        }
        
        .glass-card {
            background: var(--glass);
            backdrop-filter: blur(10px);
            -webkit-backdrop-filter: blur(10px);
            border-radius: 1rem;
            border: 1px solid rgba(255, 255, 255, 0.1);
            box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.36);
        }
        
        .status-badge {
            padding: 0.25rem 0.5rem;
            border-radius: 9999px;
            font-size: 0.75rem;
            font-weight: 600;
            text-transform: uppercase;
        }
        
        .status-watching {
            background-color: rgba(59, 130, 246, 0.2);
            color: #3b82f6;
            transition: all 0.3s ease;
        }
        
        .status-completed {
            background-color: rgba(16, 185, 129, 0.2);
            color: #10b981;
            transition: all 0.3s ease;
        }
        
        .status-reading {
            background-color: rgba(139, 92, 246, 0.2);
            color: #8b5cf6;
            transition: all 0.3s ease;
        }
        
        .status-dropped {
            background-color: rgba(239, 68, 68, 0.2);
            color: #ef4444;
            transition: all 0.3s ease;
        }
        
        .status-planned {
            background-color: rgba(249, 115, 22, 0.2);
            color: #f97316;
            transition: all 0.3s ease;
        }
        
        .media-card {
            transition: all 0.3s ease;
        }
        
        .media-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
        }
        
        .search-input:focus {
            box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.3);
        }
        
        .modal {
            transition: opacity 0.3s ease, visibility 0.3s ease;
        }
        
        .modal-content {
            transform: translateY(-20px);
            transition: transform 0.3s ease;
        }
        
        .modal.active {
            opacity: 1;
            visibility: visible;
        }
        
        .modal.active .modal-content {
            transform: translateY(0);
        }
        
        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }
        
        .fade-in {
            animation: fadeIn 0.5s ease forwards;
        }
        
        /* Custom scrollbar */
        ::-webkit-scrollbar {
            width: 8px;
        }
        
        ::-webkit-scrollbar-track {
            background: rgba(255, 255, 255, 0.05);
        }
        
        ::-webkit-scrollbar-thumb {
            background: var(--primary);
            border-radius: 4px;
        }
        
        ::-webkit-scrollbar-thumb:hover {
            background: var(--primary-dark);
        }
    </style>
</head>
<body class="min-h-screen">
    <!-- Navigation -->
    <nav class="glass-card fixed top-0 left-0 right-0 z-50 mx-auto max-w-7xl px-4 py-3 flex justify-between items-center">
        <div class="flex items-center space-x-2">
            <i class="fas fa-film text-2xl text-indigo-400"></i>
            <span class="text-xl font-bold bg-gradient-to-r from-indigo-400 to-purple-500 bg-clip-text text-transparent">MediaTracker</span>
        </div>
        
        <div class="relative w-1/3 max-w-md">
            <input type="text" placeholder="Search media..." class="search-input w-full bg-white/10 border border-white/20 rounded-full py-2 px-4 pl-10 text-white focus:outline-none focus:border-indigo-400 transition-all">
            <i class="fas fa-search absolute left-3 top-3 text-white/50"></i>
        </div>
        
        <div class="flex items-center space-x-4">
            <div id="profile-btn" class="w-10 h-10 rounded-full bg-gradient-to-br from-indigo-400 to-purple-500 flex items-center justify-center cursor-pointer hover:shadow-lg transition-all">
                <i class="fas fa-user text-white"></i>
            </div>
        </div>
    </nav>
    
    <!-- Main Content -->
    <main class="pt-24 pb-12 px-4 max-w-7xl mx-auto">
        <!-- Status Filters -->
        <div class="flex flex-wrap justify-center gap-3 mb-8">
            <button class="status-filter-btn active px-4 py-2 rounded-full bg-indigo-500/20 text-indigo-300 border border-indigo-500/30 flex items-center space-x-2 transition-all hover:bg-indigo-500/30" data-status="all">
                <i class="fas fa-list"></i>
                <span>All</span>
            </button>
            <button class="status-filter-btn px-4 py-2 rounded-full bg-blue-500/20 text-blue-300 border border-blue-500/30 flex items-center space-x-2 transition-all hover:bg-blue-500/30" data-status="watching">
                <i class="fas fa-eye"></i>
                <span>Watching</span>
            </button>
            <button class="status-filter-btn px-4 py-2 rounded-full bg-emerald-500/20 text-emerald-300 border border-emerald-500/30 flex items-center space-x-2 transition-all hover:bg-emerald-500/30" data-status="completed">
                <i class="fas fa-check-circle"></i>
                <span>Completed</span>
            </button>
            <button class="status-filter-btn px-4 py-2 rounded-full bg-purple-500/20 text-purple-300 border border-purple-500/30 flex items-center space-x-2 transition-all hover:bg-purple-500/30" data-status="reading">
                <i class="fas fa-book-open"></i>
                <span>Reading</span>
            </button>
            <button class="status-filter-btn px-4 py-2 rounded-full bg-orange-500/20 text-orange-300 border border-orange-500/30 flex items-center space-x-2 transition-all hover:bg-orange-500/30" data-status="planned">
                <i class="fas fa-bookmark"></i>
                <span>Planned</span>
            </button>
            <button class="status-filter-btn px-4 py-2 rounded-full bg-red-500/20 text-red-300 border border-red-500/30 flex items-center space-x-2 transition-all hover:bg-red-500/30" data-status="dropped">
                <i class="fas fa-times-circle"></i>
                <span>Dropped</span>
            </button>
            <button class="status-filter-btn px-4 py-2 rounded-full bg-pink-500/20 text-pink-300 border border-pink-500/30 flex items-center space-x-2 transition-all hover:bg-pink-500/30" data-status="favorite">
                <i class="fas fa-heart"></i>
                <span>Favorites</span>
            </button>
        </div>
        
        <!-- Media Grid -->
        <div id="media-grid" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
            <!-- Media cards will be dynamically inserted here -->
        </div>
        
        <!-- Empty State -->
        <div id="empty-state" class="hidden text-center py-12">
            <i class="fas fa-film text-5xl text-white/30 mb-4"></i>
            <h3 class="text-xl font-medium text-white/70 mb-2">No media found</h3>
            <p class="text-white/50 mb-4">Add some media to get started or try a different search</p>
            <button id="add-media-empty-btn" class="bg-blue-500 hover:bg-blue-600 text-white px-6 py-2 rounded-full text-sm font-medium transition-all flex items-center space-x-1">
                <i class="fas fa-plus"></i>
                <span>Add Media</span>
            </button>
        </div>
    </main>
    
    <!-- Add/Edit Media Modal -->
    <div id="media-modal" class="modal fixed inset-0 bg-black/70 flex items-center justify-center z-50 opacity-0 invisible transition-all">
        <div class="modal-content glass-card w-full max-w-md p-6 rounded-xl transform transition-all">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-xl font-bold" id="modal-title">Add New Media</h3>
                <button id="close-modal" class="text-white/50 hover:text-white transition-all">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            
            <form id="media-form" class="space-y-4">
                <input type="hidden" id="media-id">
                
                <div>
                    <label for="media-title" class="block text-sm font-medium mb-1">Title</label>
                    <input type="text" id="media-title" class="w-full bg-white/10 border border-white/20 rounded-lg py-2 px-3 text-white focus:outline-none focus:border-indigo-400 transition-all" required>
                </div>
                
                <div class="grid grid-cols-2 gap-4">
                    <div>
                        <label for="media-type" class="block text-sm font-medium mb-1">Type</label>
                        <select id="media-type" class="w-full bg-white/10 border border-white/20 rounded-lg py-2 px-3 text-white focus:outline-none focus:border-indigo-400 transition-all appearance-none" required>
                            <option value="">Select type</option>
                            <option value="movie">Movie</option>
                            <option value="tv">TV Show</option>
                            <option value="anime">Anime</option>
                            <option value="manga">Manga</option>
                            <option value="manhwa">Manhwa</option>
                            <option value="book">Book</option>
                        </select>
                    </div>
                    
                    <div>
                        <label for="media-status" class="block text-sm font-medium mb-1">Status</label>
                        <select id="media-status" class="w-full bg-white/10 border border-white/20 rounded-lg py-2 px-3 text-white focus:outline-none focus:border-indigo-400 transition-all appearance-none" required>
                            <option value="">Select status</option>
                            <option value="watching">Watching</option>
                            <option value="completed">Completed</option>
                            <option value="reading">Reading</option>
                            <option value="planned">Plan to Watch/Read</option>
                            <option value="dropped">Dropped</option>
                        </select>
                    </div>
                </div>
                
                <div>
                    <label for="media-progress" class="block text-sm font-medium mb-1">Progress</label>
                    <div class="flex items-center space-x-3">
                        <input type="number" id="media-progress" min="0" class="w-20 bg-white/10 border border-white/20 rounded-lg py-2 px-3 text-white focus:outline-none focus:border-indigo-400 transition-all">
                        <span class="text-white/70">/</span>
                        <input type="number" id="media-total" min="1" class="w-20 bg-white/10 border border-white/20 rounded-lg py-2 px-3 text-white focus:outline-none focus:border-indigo-400 transition-all">
                    </div>
                </div>
                
                <div>
                    <label for="media-rating" class="block text-sm font-medium mb-1">Rating (1-10)</label>
                    <input type="number" id="media-rating" min="0" max="10" class="w-full bg-white/10 border border-white/20 rounded-lg py-2 px-3 text-white focus:outline-none focus:border-indigo-400 transition-all">
                </div>
                
                <div class="flex items-center justify-between py-2">
                    <label for="media-favorite" class="block text-sm font-medium">Mark as Favorite</label>
                    <label class="relative inline-flex items-center cursor-pointer">
                        <input type="checkbox" id="media-favorite" class="sr-only peer">
                        <div class="w-11 h-6 bg-white/10 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-red-500/80"></div>
                    </label>
                </div>

                <div>
                    <label for="media-notes" class="block text-sm font-medium mb-1">Notes</label>
                    <textarea id="media-notes" rows="3" class="w-full bg-white/10 border border-white/20 rounded-lg py-2 px-3 text-white focus:outline-none focus:border-indigo-400 transition-all"></textarea>
                </div>
                
                <div class="flex justify-end space-x-3 pt-2">
                    <button type="button" id="cancel-media" class="px-4 py-2 rounded-lg border border-white/20 hover:bg-white/10 transition-all">
                        Cancel
                    </button>
                    <button type="submit" class="px-4 py-2 rounded-lg bg-indigo-500 hover:bg-indigo-600 text-white transition-all">
                        Save
                    </button>
                </div>
            </form>
        </div>
    </div>
    
    <!-- Profile Modal -->
    <div id="profile-modal" class="modal fixed inset-0 bg-black/70 flex items-center justify-center z-50 opacity-0 invisible transition-all">
        <div class="modal-content glass-card w-full max-w-md p-6 rounded-xl transform transition-all">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-xl font-bold">Your Profile</h3>
                <button id="close-profile-modal" class="text-white/50 hover:text-white transition-all">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            
            <div class="flex items-center space-x-4 mb-6">
                <div class="w-16 h-16 rounded-full bg-gradient-to-br from-indigo-400 to-purple-500 flex items-center justify-center">
                    <i class="fas fa-user text-2xl text-white"></i>
                </div>
                <div>
                    <h4 class="font-medium"> THE FOOL </h4>
                    <p class="text-sm text-white/60">Media Enthusiast</p>
                </div>
            </div>
            
            <div class="space-y-4">
                <div class="flex justify-between items-center">
                    <span class="text-white/70">Total Media</span>
                    <span class="font-medium" id="total-media-count">0</span>
                </div>
                
                <div class="flex justify-between items-center">
                    <span class="text-white/70">Completed</span>
                    <span class="font-medium" id="completed-count">0</span>
                </div>
                
                <div class="flex justify-between items-center">
                    <span class="text-white/70">Currently Watching</span>
                    <span class="font-medium" id="watching-count">0</span>
                </div>
                
                <div class="flex justify-between items-center">
                    <span class="text-white/70">Currently Reading</span>
                    <span class="font-medium" id="reading-count">0</span>
                </div>
                
                <div class="flex justify-between items-center">
                    <span class="text-white/70">Planned</span>
                    <span class="font-medium" id="planned-count">0</span>
                </div>
                
                <div class="flex justify-between items-center">
                    <span class="text-white/70">Dropped</span>
                    <span class="font-medium" id="dropped-count">0</span>
                </div>
            </div>
            
        </div>
    </div>
    
    <!-- Floating Add Button -->
    <button id="add-media-btn" class="fixed bottom-6 right-6 w-14 h-14 rounded-full bg-blue-500 hover:bg-blue-600 text-white text-2xl flex items-center justify-center shadow-lg transition-all z-40">
        <i class="fas fa-plus"></i>
    </button>

    <!-- Delete Confirmation Modal -->
    <div id="delete-modal" class="modal fixed inset-0 bg-black/70 flex items-center justify-center z-50 opacity-0 invisible transition-all">
        <div class="modal-content glass-card w-full max-w-md p-6 rounded-xl transform transition-all">
            <div class="text-center">
                <i class="fas fa-exclamation-triangle text-4xl text-red-400 mb-4"></i>
                <h3 class="text-xl font-bold mb-2">Delete Media</h3>
                <p class="text-white/70 mb-6">Are you sure you want to delete this media entry? This action cannot be undone.</p>
                
                <div class="flex justify-center space-x-3">
                    <button id="cancel-delete" class="px-4 py-2 rounded-lg border border-white/20 hover:bg-white/10 transition-all">
                        Cancel
                    </button>
                    <button id="confirm-delete" class="px-4 py-2 rounded-lg bg-red-500 hover:bg-red-600 text-white transition-all">
                        Delete
                    </button>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Sample data
        let mediaData = [
            {
                id: 1,
                title: "Attack on Titan",
                type: "anime",
                status: "completed",
                progress: 88,
                total: 88,
                rating: 10,
                notes: "One of the best anime ever made. The story is incredible.",
                favorite: true
            },
            {
                id: 2,
                title: "One Piece",
                type: "manga",
                status: "reading",
                progress: 1045,
                total: 1085,
                rating: 9,
                notes: "The Wano arc is amazing!"
            },
            {
                id: 3,
                title: "The Matrix",
                type: "movie",
                status: "completed",
                progress: 1,
                total: 1,
                rating: 9,
                notes: "Classic sci-fi that changed cinema."
            },
            {
                id: 4,
                title: "Breaking Bad",
                type: "tv",
                status: "completed",
                progress: 62,
                total: 62,
                rating: 10,
                notes: "Masterpiece of television."
            },
            {
                id: 5,
                title: "Solo Leveling",
                type: "manhwa",
                status: "reading",
                progress: 110,
                total: 179,
                rating: 8,
                notes: "Art is amazing, story is engaging."
            },
            {
                id: 6,
                title: "Dune",
                type: "book",
                status: "planned",
                progress: 0,
                total: 412,
                rating: 0,
                notes: "Want to read before watching the movie."
            },
            {
                id: 7,
                title: "Demon Slayer",
                type: "anime",
                status: "watching",
                progress: 18,
                total: 26,
                rating: 8,
                notes: "Animation is stunning."
            },
            {
                id: 8,
                title: "The Witcher",
                type: "tv",
                status: "dropped",
                progress: 5,
                total: 8,
                rating: 6,
                notes: "Didn't like the changes from the books."
            }
        ];
        
        // DOM Elements
        const mediaGrid = document.getElementById('media-grid');
        const emptyState = document.getElementById('empty-state');
        const searchInput = document.querySelector('.search-input');
        const statusFilterBtns = document.querySelectorAll('.status-filter-btn');
        const addMediaBtn = document.getElementById('add-media-btn');
        const addMediaEmptyBtn = document.getElementById('add-media-empty-btn');
        const profileBtn = document.getElementById('profile-btn');
        
        // Modal Elements
        const mediaModal = document.getElementById('media-modal');
        const modalTitle = document.getElementById('modal-title');
        const mediaForm = document.getElementById('media-form');
        const mediaIdInput = document.getElementById('media-id');
        const mediaTitleInput = document.getElementById('media-title');
        const mediaTypeInput = document.getElementById('media-type');
        const mediaStatusInput = document.getElementById('media-status');
        const mediaProgressInput = document.getElementById('media-progress');
        const mediaTotalInput = document.getElementById('media-total');
        const mediaRatingInput = document.getElementById('media-rating');
        const mediaNotesInput = document.getElementById('media-notes');
        const closeModalBtn = document.getElementById('close-modal');
        const cancelMediaBtn = document.getElementById('cancel-media');
        
        const profileModal = document.getElementById('profile-modal');
        const closeProfileModalBtn = document.getElementById('close-profile-modal');
        
        const deleteModal = document.getElementById('delete-modal');
        const cancelDeleteBtn = document.getElementById('cancel-delete');
        const confirmDeleteBtn = document.getElementById('confirm-delete');
        
        // State
        let currentFilter = 'all';
        let currentSearch = '';
        let mediaToDelete = null;
        
        // Initialize
        function init() {
            renderMedia();
            updateStats();
            setupEventListeners();
        }
        
        // Event Listeners
        function setupEventListeners() {
            // Search
            searchInput.addEventListener('input', (e) => {
                currentSearch = e.target.value.toLowerCase();
                renderMedia();
            });
            
            // Status filters
            statusFilterBtns.forEach(btn => {
                btn.addEventListener('click', () => {
                    statusFilterBtns.forEach(b => b.classList.remove('active'));
                    btn.classList.add('active');
                    currentFilter = btn.dataset.status;
                    renderMedia();
                });
            });
            
            // Add media buttons
            addMediaBtn.addEventListener('click', openAddMediaModal);
            addMediaEmptyBtn.addEventListener('click', openAddMediaModal);
            
            // Profile button
            profileBtn.addEventListener('click', () => {
                profileModal.classList.add('active');
            });
            
            // Modal buttons
            closeModalBtn.addEventListener('click', closeMediaModal);
            cancelMediaBtn.addEventListener('click', closeMediaModal);
            closeProfileModalBtn.addEventListener('click', () => {
                profileModal.classList.remove('active');
            });
            
            // Form submission
            mediaForm.addEventListener('submit', handleFormSubmit);
            
            // Delete modal
            cancelDeleteBtn.addEventListener('click', () => {
                deleteModal.classList.remove('active');
                mediaToDelete = null;
            });
            
            confirmDeleteBtn.addEventListener('click', () => {
                if (mediaToDelete) {
                    deleteMedia(mediaToDelete);
                    deleteModal.classList.remove('active');
                    mediaToDelete = null;
                }
            });
        }
        
        // Render media
        function renderMedia() {
            // Filter media based on search and status
            let filteredMedia = mediaData.filter(media => {
                const matchesSearch = media.title.toLowerCase().includes(currentSearch);
                const matchesFilter = currentFilter === 'all' 
                    || media.status === currentFilter
                    || (currentFilter === 'favorite' && media.favorite);
                return matchesSearch && matchesFilter;
            });
            
            // Clear grid
            mediaGrid.innerHTML = '';
            
            // Show empty state if no media
            if (filteredMedia.length === 0) {
                emptyState.classList.remove('hidden');
                return;
            } else {
                emptyState.classList.add('hidden');
            }
            
            // Add media cards
            filteredMedia.forEach(media => {
                const mediaCard = createMediaCard(media);
                mediaGrid.appendChild(mediaCard);
            });
        }
        
        // Create media card HTML
        function createMediaCard(media) {
            const card = document.createElement('div');
            card.className = 'media-card glass-card p-4 rounded-xl fade-in';
            card.dataset.id = media.id;
            
            // Get icon based on type
            let typeIcon = '';
            let typeClass = '';
            switch(media.type) {
                case 'movie':
                    typeIcon = 'fas fa-film';
                    typeClass = 'text-blue-400';
                    break;
                case 'tv':
                    typeIcon = 'fas fa-tv';
                    typeClass = 'text-purple-400';
                    break;
                case 'anime':
                    typeIcon = 'fas fa-dragon';
                    typeClass = 'text-pink-400';
                    break;
                case 'manga':
                case 'manhwa':
                    typeIcon = 'fas fa-book';
                    typeClass = 'text-red-400';
                    break;
                case 'book':
                    typeIcon = 'fas fa-book-open';
                    typeClass = 'text-yellow-400';
                    break;
                default:
                    typeIcon = 'fas fa-question';
            }
            
            // Status badge
            let statusBadge = '';
            let statusText = '';
            switch(media.status) {
                case 'watching':
                    statusBadge = 'status-watching';
                    statusText = 'Watching';
                    break;
                case 'completed':
                    statusBadge = 'status-completed';
                    statusText = 'Completed';
                    break;
                case 'reading':
                    statusBadge = 'status-reading';
                    statusText = 'Reading';
                    break;
                case 'planned':
                    statusBadge = 'status-planned';
                    statusText = 'Planned';
                    break;
                case 'dropped':
                    statusBadge = 'status-dropped';
                    statusText = 'Dropped';
                    break;
            }
            
            // Progress bar
            const progressPercentage = media.total > 0 ? Math.min(100, (media.progress / media.total) * 100) : 0;
            
            card.innerHTML = `
                <div class="flex justify-between items-start mb-3">
                    <div class="flex items-center space-x-2">
                        <i class="${typeIcon} ${typeClass}"></i>
                        <span class="text-sm font-medium capitalize">${media.type}</span>
                    </div>
                    <span class="${statusBadge} status-badge">${statusText}</span>
                </div>
                
                <div class="flex items-center space-x-2 mb-2">
                    <h3 class="text-lg font-bold line-clamp-1 flex-grow">${media.title}</h3>
                    ${media.favorite ? '<i class="fas fa-heart text-red-400"></i>' : ''}
                </div>
                
                <div class="mb-3">
                    <div class="flex justify-between text-xs text-white/70 mb-1">
                        <span>Progress</span>
                        <span>${media.progress}/${media.total}</span>
                    </div>
                    <div class="w-full bg-white/10 rounded-full h-2">
                        <div class="bg-indigo-500 h-2 rounded-full" style="width: ${progressPercentage}%"></div>
                    </div>
                </div>
                
                ${media.rating > 0 ? `
                <div class="flex items-center mb-3">
                    <div class="flex">
                        ${Array.from({length: 5}, (_, i) => `
                            <i class="fas fa-star ${i < Math.floor(media.rating / 2) ? 'text-yellow-400' : 'text-white/20'}"></i>
                        `).join('')}
                    </div>
                    <span class="ml-2 text-sm font-medium">${media.rating}/10</span>
                </div>
                ` : ''}
                
                ${media.notes ? `
                <p class="text-sm text-white/70 line-clamp-2 mb-4">${media.notes}</p>
                ` : ''}
                
                <div class="flex justify-end space-x-2">
                    <button class="edit-btn px-3 py-1 rounded-lg bg-white/5 hover:bg-white/10 transition-all text-sm" data-id="${media.id}">
                        <i class="fas fa-edit"></i>
                    </button>
                    <button class="delete-btn px-3 py-1 rounded-lg bg-red-500/20 hover:bg-red-500/30 transition-all text-sm" data-id="${media.id}">
                        <i class="fas fa-trash"></i>
                    </button>
                </div>
            `;
            
            // Add event listeners to buttons
            card.querySelector('.edit-btn').addEventListener('click', () => openEditMediaModal(media.id));
            card.querySelector('.delete-btn').addEventListener('click', () => confirmDeleteMedia(media.id));
            
            return card;
        }
        
        // Open add media modal
        function openAddMediaModal() {
            mediaForm.reset();
            mediaIdInput.value = '';
            modalTitle.textContent = 'Add New Media';
            mediaModal.classList.add('active');
        }
        
        // Open edit media modal
        function openEditMediaModal(id) {
            const media = mediaData.find(m => m.id === id);
            if (!media) return;
            
            mediaIdInput.value = media.id;
            mediaTitleInput.value = media.title;
            mediaTypeInput.value = media.type;
            mediaStatusInput.value = media.status;
            mediaProgressInput.value = media.progress;
            mediaTotalInput.value = media.total;
            mediaRatingInput.value = media.rating;
            mediaNotesInput.value = media.notes || '';
            document.getElementById('media-favorite').checked = media.favorite || false;
            
            modalTitle.textContent = 'Edit Media';
            mediaModal.classList.add('active');
        }
        
        // Close media modal
        function closeMediaModal() {
            mediaModal.classList.remove('active');
        }
        
        // Handle form submission
        function handleFormSubmit(e) {
            e.preventDefault();
            
            const id = mediaIdInput.value ? parseInt(mediaIdInput.value) : Date.now();
            const media = {
                id,
                title: mediaTitleInput.value,
                type: mediaTypeInput.value,
                status: mediaStatusInput.value,
                progress: parseInt(mediaProgressInput.value) || 0,
                total: parseInt(mediaTotalInput.value) || 1,
                rating: parseInt(mediaRatingInput.value) || 0,
                notes: mediaNotesInput.value,
                favorite: document.getElementById('media-favorite').checked
            };
            
            if (media.progress > media.total) {
                media.progress = media.total;
            }
            
            // Update or add media
            const existingIndex = mediaData.findIndex(m => m.id === id);
            if (existingIndex >= 0) {
                mediaData[existingIndex] = media;
            } else {
                mediaData.push(media);
            }
            
            closeMediaModal();
            renderMedia();
            updateStats();
        }
        
        // Confirm delete media
        function confirmDeleteMedia(id) {
            mediaToDelete = id;
            deleteModal.classList.add('active');
        }
        
        // Delete media
        function deleteMedia(id) {
            mediaData = mediaData.filter(m => m.id !== id);
            renderMedia();
            updateStats();
        }
        
        // Update stats in profile
        function updateStats() {
            document.getElementById('total-media-count').textContent = mediaData.length;
            document.getElementById('completed-count').textContent = mediaData.filter(m => m.status === 'completed').length;
            document.getElementById('watching-count').textContent = mediaData.filter(m => m.status === 'watching').length;
            document.getElementById('reading-count').textContent = mediaData.filter(m => m.status === 'reading').length;
            document.getElementById('planned-count').textContent = mediaData.filter(m => m.status === 'planned').length;
            document.getElementById('dropped-count').textContent = mediaData.filter(m => m.status === 'dropped').length;
        }
        
        // Initialize the app
        init();
    </script>
</html>
