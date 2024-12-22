<script>
    import { onMount } from 'svelte';
    import axios from 'axios';

    // just for commit
    let url = '';
    let formats = [];
    let title = '';
    let errorMessage = '';
    let loading = false;
    let showAudio = true;  // Controls the visibility of the Audio section
    let showVideo = false; // Controls the visibility of the Video section
    let thumbnailUrl = ''; // Stores the video thumbnail URL

    // Fetch video info from the API
    const fetchVideoInfo = async () => {
        if (!url) {
            errorMessage = 'Please enter a valid URL.';
            return;
        }
        errorMessage = '';
        formats = [];
        loading = true;

        try {
            const response = await axios.get('https://d30d-103-151-43-62.ngrok-free.app/api/video-info/', {
                params: { url },
            });
            title = response.data.title;
            formats = response.data.formats;
            thumbnailUrl = response.data.thumbnail; // Assuming the thumbnail URL is part of the response
        } catch (error) {
            errorMessage = error.response?.data?.error || 'An error occurred.';
        } finally {
            loading = false;
        }
    };

    // Toggle between Audio and Video sections
    const toggleSection = (section) => {
        if (section === 'audio') {
            showAudio = true;
            showVideo = false;
        } else if (section === 'video') {
            showAudio = false;
            showVideo = true;
        }
    };

    // Trigger the download process
    const downloadFile = (url) => {
        const link = document.createElement('a');
        link.href = url;
        link.download = url.split('/').pop(); // Set the download file name based on the URL
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    };
</script>

<main>
    <div class="container">
        <div class="card">
            <h1>YouTube Video Downloader</h1>
            <div class="input-container">
                <input
                    type="text"
                    placeholder="Paste YouTube URL"
                    bind:value={url}
                />
                <button on:click={fetchVideoInfo} disabled={loading}>
                    {loading ? 'Loading...' : 'Get Video Info'}
                </button>
            </div>

            <!-- Error message -->
            {#if errorMessage}
                <p class="error">{errorMessage}</p>
            {/if}

            <!-- Display video title and thumbnail -->
            {#if title}
                <div class="video-header">
                    {#if thumbnailUrl}
                        <img src={thumbnailUrl} alt="Thumbnail" class="thumbnail" />
                    {/if}
                    <h2>{title}</h2>
                </div>
            {/if}

            <!-- Display audio and video sections -->
            {#if formats.length > 0}
                <div class="tabs">
                    <button class="tab" on:click={() => toggleSection('audio')} class:selected={showAudio}>Audio</button>
                    <button class="tab" on:click={() => toggleSection('video')} class:selected={showVideo}>Video</button>
                </div>

                <!-- Audio Section -->
                {#if showAudio}
                    <div class="audio-video-box">
                        <div class="audio-box">
                            <h3>Audio</h3>
                            {#each formats as format (format.format_id)}
                                {#if format.type === 'audio' && format.ext === 'm4a'}
                                    <div class="audio-item">
                                        <p>{format.resolution}</p>
                                        <button on:click={() => downloadFile(format.url)}>
                                            Download {format.ext}
                                        </button>
                                    </div>
                                {/if}
                            {/each}
                        </div>
                    </div>
                {/if}

                <!-- Video Section -->
                {#if showVideo}
                    <div class="audio-video-box">
                        <div class="video-box">
                            <h3>Video</h3>
                            {#each formats as format (format.format_id)}
                                {#if format.type === 'video'}
                                    <div class="video-item">
                                        <p>{format.resolution}</p>
                                        <button on:click={() => downloadFile(format.url)}>
                                            Download {format.ext}
                                        </button>
                                    </div>
                                {/if}
                            {/each}
                        </div>
                    </div>
                {/if}
            {/if}
        </div>
    </div>
</main>



<style>
    * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }

    body {
        font-family: Arial, sans-serif;
        background-color: #f9f9f9;
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 100vh;
    }

    .container {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 100%;
        padding: 1rem;
    }
	.thumbnail {
        width: 480px;
        height: 320px;
        margin-right: 1rem;
        border-radius: 8px;
        object-fit: contain;
    }
	
    .card {
        background-color: white;
        border-radius: 12px;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
        width: 100%;
        max-width: 800px;
        padding: 2rem;
        text-align: center;
    }

    h1 {
        font-size: 2rem;
        color: #333;
        margin-bottom: 1rem;
    }

    .input-container {
        margin-bottom: 1rem;
    }

    input {
        padding: 0.8rem;
        width: 70%;
        margin-right: 1rem;
        border-radius: 5px;
        border: 1px solid #ccc;
    }

    button {
        padding: 0.8rem 1rem;
        background-color: #4CAF50;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
    }

    button:disabled {
        background-color: #ccc;
        cursor: not-allowed;
    }

    .error {
        color: red;
        margin-top: 1rem;
    }

    .video-header {
        display: flex;
        justify-content: center;
        align-items: center;
        margin-bottom: 1rem;
    }

    .thumbnail {
        width: 240px;
        height: 160px;
        margin-right: 1rem;
        border-radius: 8px;
        object-fit: cover;
    }

    h2 {
        font-size: 1.5rem;
        color: #333;
    }

    .tabs {
        margin-top: 1rem;
        display: flex;
        justify-content: center;
        gap: 1rem;
    }

    .tab {
        padding: 0.8rem 1.5rem;
        border: none;
        background-color: #f0f0f0;
        cursor: pointer;
        border-radius: 5px;
        font-weight: bold;
    }

    .tab.selected {
        background-color: #4CAF50;
        color: white;
    }

    .audio-video-box {
        display: flex;
		justify-content: center;
		align-items: center;
        gap: 2rem;
        margin-top: 1rem;
        justify-content: space-between;
    }

    .audio-box, .video-box {
		margin-left: 200px;
        width: 48%;
        padding: 1rem;
        border: 1px solid #ddd;
        border-radius: 8px;
        box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
    }

    .audio-item, .video-item {
        margin-bottom: 1rem;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    a {
        color: #5aa53d;
        text-decoration: none;
        font-weight: bold;
    }

    a:hover {
        text-decoration: underline;
    }
</style>
