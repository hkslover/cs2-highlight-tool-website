<template>
  <footer class="site-footer">
    <div class="container">
      <div class="footer-top">
        <div class="footer-brand">
          <h3 class="footer-logo">CS2 Highlight Tool</h3>
          <p class="footer-tagline">开源 CS2 高光集锦一键制作工具</p>
        </div>
        <div class="footer-links">
          <a href="https://github.com/hkslover/cs2-highlight-tool" target="_blank" class="footer-link">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor"><path d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"/></svg>
            GitHub
          </a>
        </div>
      </div>

      <!-- <div class="footer-tech">
        <span class="tech-label">Built with</span>
        <div class="tech-tags">
          <span class="tech-tag">Go</span>
          <span class="tech-tag">Wails</span>
          <span class="tech-tag">Vue 3</span>
          <span class="tech-tag">TypeScript</span>
          <span class="tech-tag">HLAE</span>
          <span class="tech-tag">FFmpeg</span>
        </div>
      </div> -->

      <!-- <div class="footer-bottom">
        <p>Made with passion for the CS2 community</p>
      </div> -->
    </div>
  </footer>
</template>

<style scoped>
.site-footer {
  padding: 80px 0 40px;
  border-top: 1px solid var(--color-border);
  background: var(--color-surface);
  scroll-snap-align: end;
}

.footer-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 40px;
}

.footer-logo {
  font-size: 20px;
  font-weight: 800;
  color: var(--color-text);
  margin-bottom: 6px;
}

.footer-tagline {
  font-size: 14px;
  color: var(--color-text-muted);
}

.footer-links {
  display: flex;
  gap: 16px;
}

.footer-link {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  font-weight: 600;
  color: var(--color-text-secondary);
  padding: 8px 16px;
  border-radius: 8px;
  border: 1px solid var(--color-border);
  transition: all 0.2s;
}

.footer-link:hover {
  border-color: var(--color-primary);
  color: var(--color-accent);
}

.footer-tech {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 40px;
  flex-wrap: wrap;
}

.tech-label {
  font-size: 12px;
  color: var(--color-text-muted);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.tech-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.tech-tag {
  padding: 4px 12px;
  border-radius: 999px;
  background: rgba(47, 148, 98, 0.08);
  border: 1px solid rgba(47, 148, 98, 0.2);
  color: var(--color-accent);
  font-size: 12px;
  font-weight: 600;
}

.footer-bottom {
  padding-top: 24px;
  border-top: 1px solid var(--color-border);
  text-align: center;
  font-size: 13px;
  color: var(--color-text-muted);
}

@media (max-width: 768px) {
  .footer-top {
    flex-direction: column;
    gap: 20px;
  }
}
</style>
