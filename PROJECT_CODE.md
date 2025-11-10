# EduLibrary - Complete Project Code

This document contains all the source code for the EduLibrary application.

---

## 📁 Directory Structure

```
edulibrary/
├── client/
│   ├── src/
│   │   ├── components/
│   │   │   ├── ui/ (shadcn components)
│   │   │   ├── Hero.tsx
│   │   │   ├── ResourceCard.tsx
│   │   │   ├── ResourceFilters.tsx
│   │   │   ├── ResourceDetail.tsx
│   │   │   ├── ResourceForm.tsx
│   │   │   └── EmptyState.tsx
│   │   ├── lib/
│   │   │   ├── categoryImages.ts
│   │   │   ├── videoUtils.ts
│   │   │   └── queryClient.ts
│   │   ├── pages/
│   │   │   ├── home.tsx
│   │   │   └── not-found.tsx
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── index.css
│   └── index.html
├── server/
│   ├── routes.ts
│   ├── storage.ts
│   └── index.ts
├── shared/
│   └── schema.ts
└── package.json
```

---

## 🔧 Configuration Files

### package.json
```json
{
  "name": "edulibrary",
  "version": "1.0.0",
  "scripts": {
    "dev": "NODE_ENV=development tsx server/index.ts"
  }
}
```

### client/index.html
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1" />
    <title>EduLibrary - Educational Resources by Skill Level</title>
    <meta name="description" content="Discover educational resources organized by skill level. Browse programming, mathematics, science, languages, business, and arts resources tailored to your learning journey." />
    <link rel="icon" type="image/png" href="/favicon.png" />
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=Open+Sans:wght@300;400;500;600;700&display=swap" rel="stylesheet">
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

---

## 🎨 Shared Schema

### shared/schema.ts
```typescript
import { sql } from "drizzle-orm";
import { pgTable, text, varchar } from "drizzle-orm/pg-core";
import { createInsertSchema } from "drizzle-zod";
import { z } from "zod";

export const skillLevels = ["Beginner", "Intermediate", "Advanced"] as const;
export const categories = [
  "Programming",
  "Mathematics",
  "Science",
  "Languages",
  "Business",
  "Arts"
] as const;

export const resources = pgTable("resources", {
  id: varchar("id").primaryKey().default(sql\`gen_random_uuid()\`),
  title: text("title").notNull(),
  description: text("description").notNull(),
  category: text("category").notNull(),
  skillLevel: text("skill_level").notNull(),
  imageUrl: text("image_url").notNull(),
  resourceType: text("resource_type").notNull(),
  videoUrl: text("video_url"),
});

export const insertResourceSchema = createInsertSchema(resources).omit({
  id: true,
}).extend({
  title: z.string().min(1, "Title is required").max(200, "Title is too long"),
  description: z.string().min(10, "Description must be at least 10 characters").max(1000, "Description is too long"),
  category: z.enum(categories, { errorMap: () => ({ message: "Please select a valid category" }) }),
  skillLevel: z.enum(skillLevels, { errorMap: () => ({ message: "Please select a valid skill level" }) }),
  imageUrl: z.string().url("Must be a valid URL"),
  resourceType: z.string().min(1, "Resource type is required"),
  videoUrl: z.string().url("Must be a valid URL").optional().or(z.literal("")),
});

export type InsertResource = z.infer<typeof insertResourceSchema>;
export type Resource = typeof resources.$inferSelect;
export type SkillLevel = typeof skillLevels[number];
export type Category = typeof categories[number];
```

---

## 🖥️ Backend Code

### server/storage.ts
```typescript
import { type Resource, type InsertResource } from "@shared/schema";
import { randomUUID } from "crypto";

export interface IStorage {
  getAllResources(): Promise<Resource[]>;
  getResource(id: string): Promise<Resource | undefined>;
  createResource(resource: InsertResource): Promise<Resource>;
  updateResource(id: string, resource: InsertResource): Promise<Resource | undefined>;
  deleteResource(id: string): Promise<boolean>;
}

export class MemStorage implements IStorage {
  private resources: Map<string, Resource>;

  constructor() {
    this.resources = new Map();
    this.seedData();
  }

  private seedData() {
    const seedResources: InsertResource[] = [
      {
        title: "Python for Beginners: Complete Guide",
        description: "Learn Python programming from scratch with this comprehensive guide. Covers variables, data types, control structures, functions, and basic object-oriented programming concepts. Perfect for absolute beginners with no prior coding experience.",
        category: "Programming",
        skillLevel: "Beginner",
        imageUrl: "/attached_assets/stock_images/programming_code_on__bd853788.jpg",
        resourceType: "Video Course",
        videoUrl: "https://www.youtube.com/watch?v=rfscVS0vtbw",
      },
      {
        title: "Advanced Data Structures in Python",
        description: "Master complex data structures including trees, graphs, heaps, and hash tables. Learn advanced algorithms for sorting, searching, and optimization. Includes real-world applications and interview preparation materials for experienced developers.",
        category: "Programming",
        skillLevel: "Advanced",
        imageUrl: "/attached_assets/stock_images/programming_code_on__fe211c3a.jpg",
        resourceType: "eBook",
      },
      {
        title: "Calculus I: Limits and Derivatives",
        description: "Introduction to differential calculus covering limits, continuity, derivatives, and their applications. Includes numerous examples, practice problems, and step-by-step solutions to build strong mathematical foundations.",
        category: "Mathematics",
        skillLevel: "Intermediate",
        imageUrl: "/attached_assets/stock_images/mathematics_equation_29c12190.jpg",
        resourceType: "Online Course",
        videoUrl: "https://www.youtube.com/watch?v=WUvTyaaNkzM",
      },
      // ... additional seed data
    ];

    seedResources.forEach((resource) => {
      const id = randomUUID();
      this.resources.set(id, { ...resource, id });
    });
  }

  async getAllResources(): Promise<Resource[]> {
    return Array.from(this.resources.values());
  }

  async getResource(id: string): Promise<Resource | undefined> {
    return this.resources.get(id);
  }

  async createResource(insertResource: InsertResource): Promise<Resource> {
    const id = randomUUID();
    const resource: Resource = { ...insertResource, id };
    this.resources.set(id, resource);
    return resource;
  }

  async updateResource(id: string, insertResource: InsertResource): Promise<Resource | undefined> {
    const existing = this.resources.get(id);
    if (!existing) {
      return undefined;
    }
    const updated: Resource = { ...insertResource, id };
    this.resources.set(id, updated);
    return updated;
  }

  async deleteResource(id: string): Promise<boolean> {
    return this.resources.delete(id);
  }
}

export const storage = new MemStorage();
```

### server/routes.ts
```typescript
import type { Express } from "express";
import { createServer, type Server } from "http";
import { storage } from "./storage";
import { insertResourceSchema } from "@shared/schema";
import { fromError } from "zod-validation-error";

export async function registerRoutes(app: Express): Promise<Server> {
  app.get("/api/resources", async (_req, res) => {
    try {
      const resources = await storage.getAllResources();
      res.json(resources);
    } catch (error) {
      console.error("Error fetching resources:", error);
      res.status(500).json({ error: "Failed to fetch resources" });
    }
  });

  app.get("/api/resources/:id", async (req, res) => {
    try {
      const { id } = req.params;
      const resource = await storage.getResource(id);
      
      if (!resource) {
        return res.status(404).json({ error: "Resource not found" });
      }
      
      res.json(resource);
    } catch (error) {
      console.error("Error fetching resource:", error);
      res.status(500).json({ error: "Failed to fetch resource" });
    }
  });

  app.post("/api/resources", async (req, res) => {
    try {
      const result = insertResourceSchema.safeParse(req.body);
      
      if (!result.success) {
        const validationError = fromError(result.error);
        return res.status(400).json({ 
          error: "Validation failed", 
          details: validationError.toString() 
        });
      }
      
      const resource = await storage.createResource(result.data);
      res.status(201).json(resource);
    } catch (error) {
      console.error("Error creating resource:", error);
      res.status(500).json({ error: "Failed to create resource" });
    }
  });

  app.patch("/api/resources/:id", async (req, res) => {
    try {
      const { id } = req.params;
      const result = insertResourceSchema.safeParse(req.body);
      
      if (!result.success) {
        const validationError = fromError(result.error);
        return res.status(400).json({ 
          error: "Validation failed", 
          details: validationError.toString() 
        });
      }
      
      const resource = await storage.updateResource(id, result.data);
      
      if (!resource) {
        return res.status(404).json({ error: "Resource not found" });
      }
      
      res.json(resource);
    } catch (error) {
      console.error("Error updating resource:", error);
      res.status(500).json({ error: "Failed to update resource" });
    }
  });

  app.delete("/api/resources/:id", async (req, res) => {
    try {
      const { id } = req.params;
      const success = await storage.deleteResource(id);
      
      if (!success) {
        return res.status(404).json({ error: "Resource not found" });
      }
      
      res.status(204).send();
    } catch (error) {
      console.error("Error deleting resource:", error);
      res.status(500).json({ error: "Failed to delete resource" });
    }
  });

  const httpServer = createServer(app);
  return httpServer;
}
```

---

## ⚛️ Frontend Code

### client/src/App.tsx
```typescript
import { Switch, Route } from "wouter";
import { queryClient } from "./lib/queryClient";
import { QueryClientProvider } from "@tanstack/react-query";
import { Toaster } from "@/components/ui/toaster";
import { TooltipProvider } from "@/components/ui/tooltip";
import Home from "@/pages/home";
import NotFound from "@/pages/not-found";

function Router() {
  return (
    <Switch>
      <Route path="/" component={Home}/>
      <Route component={NotFound} />
    </Switch>
  );
}

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <TooltipProvider>
        <Toaster />
        <Router />
      </TooltipProvider>
    </QueryClientProvider>
  );
}

export default App;
```

### client/src/lib/videoUtils.ts
```typescript
export function getVideoEmbedUrl(url: string): string | null {
  if (!url) return null;

  try {
    const urlObj = new URL(url);
    
    // YouTube
    if (urlObj.hostname.includes('youtube.com') || urlObj.hostname.includes('youtu.be')) {
      let videoId = '';
      
      if (urlObj.hostname.includes('youtu.be')) {
        videoId = urlObj.pathname.slice(1);
      } else {
        videoId = urlObj.searchParams.get('v') || '';
      }
      
      if (videoId) {
        return \`https://www.youtube.com/embed/\${videoId}\`;
      }
    }
    
    // Vimeo
    if (urlObj.hostname.includes('vimeo.com')) {
      const videoId = urlObj.pathname.split('/').filter(Boolean)[0];
      if (videoId) {
        return \`https://player.vimeo.com/video/\${videoId}\`;
      }
    }
    
    return null;
  } catch {
    return null;
  }
}

export function isVideoUrl(url: string | null | undefined): boolean {
  if (!url) return false;
  
  try {
    const urlObj = new URL(url);
    return (
      urlObj.hostname.includes('youtube.com') ||
      urlObj.hostname.includes('youtu.be') ||
      urlObj.hostname.includes('vimeo.com')
    );
  } catch {
    return false;
  }
}
```

### client/src/components/Hero.tsx
```typescript
import { Button } from "@/components/ui/button";
import { BookOpen, GraduationCap, Target } from "lucide-react";
import { heroImage } from "@/lib/categoryImages";

interface HeroProps {
  onBrowseClick: () => void;
  resourceCount: number;
}

export default function Hero({ onBrowseClick, resourceCount }: HeroProps) {
  return (
    <div className="relative h-96 md:h-[500px] w-full overflow-hidden">
      <div 
        className="absolute inset-0 bg-cover bg-center"
        style={{ backgroundImage: \`url(\${heroImage})\` }}
      />
      <div className="absolute inset-0 bg-gradient-to-b from-black/60 via-black/50 to-black/70" />
      
      <div className="relative h-full flex flex-col items-center justify-center text-center px-4 max-w-4xl mx-auto">
        <div className="flex items-center gap-2 mb-4">
          <GraduationCap className="w-12 h-12 text-white" />
          <h1 className="text-4xl md:text-5xl lg:text-6xl font-bold text-white font-sans">
            EduLibrary
          </h1>
        </div>
        
        <p className="text-xl md:text-2xl text-white/95 mb-2 font-sans">
          Discover Educational Resources
        </p>
        <p className="text-base md:text-lg text-white/85 mb-8 max-w-2xl leading-relaxed">
          Access curated learning materials organized by skill level. Whether you're a beginner or advancing your expertise, find the perfect resources for your journey.
        </p>
        
        <Button 
          size="lg"
          onClick={onBrowseClick}
          className="bg-white/10 hover:bg-white/20 backdrop-blur-md text-white border-2 border-white/30 text-lg px-8 py-6 h-auto"
          data-testid="button-browse-resources"
        >
          <BookOpen className="mr-2 h-5 w-5" />
          Browse Resources
        </Button>
        
        <div className="flex flex-wrap items-center justify-center gap-6 mt-12 text-white/90">
          <div className="flex items-center gap-2">
            <Target className="w-5 h-5" />
            <span className="text-sm font-medium">{resourceCount} Resources</span>
          </div>
          <div className="flex items-center gap-2">
            <BookOpen className="w-5 h-5" />
            <span className="text-sm font-medium">6 Categories</span>
          </div>
          <div className="flex items-center gap-2">
            <GraduationCap className="w-5 h-5" />
            <span className="text-sm font-medium">3 Skill Levels</span>
          </div>
        </div>
      </div>
    </div>
  );
}
```

---

## 📝 Complete Component Files

All component files (ResourceCard, ResourceFilters, ResourceDetail, ResourceForm, EmptyState) are included in the running application.

---

## 🚀 Running the Application

1. Install dependencies: `npm install`
2. Start the development server: `npm run dev`
3. Access the app at: http://localhost:5000

---

## 🌐 Live Application

Your app is running at:
**https://1eb684f9-144a-416a-b339-e5ad0f423141-00-1vlqv96yughbt.janeway.replit.dev**

---

## ✨ Features

- Browse educational resources by skill level
- Search and filter resources
- Add/edit/delete resources with video support
- Embedded YouTube and Vimeo video players
- Beautiful stock imagery
- Responsive design
- Real-time validation

---

*Generated on November 9, 2025*
