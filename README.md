import React, { useCallback, useEffect, useMemo, useRef, useState } from "react";
import { motion } from "framer-motion";
import { createClient } from "@supabase/supabase-js";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Badge } from "@/components/ui/badge";
import { Avatar, AvatarFallback } from "@/components/ui/avatar";
import { ScrollArea } from "@/components/ui/scroll-area";
import { Alert, AlertDescription } from "@/components/ui/alert";
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger } from "@/components/ui/dialog";
import { MessageCircle, LogIn, LogOut, Reply, RefreshCcw, Users, Sparkles, Plus } from "lucide-react";

const SUPABASE_URL = typeof import.meta !== "undefined" && import.meta.env ? import.meta.env.VITE_SUPABASE_URL : "https://xvtmssiyqlmcevgkadth.supabase.co";
const SUPABASE_ANON_KEY = typeof import.meta !== "undefined" && import.meta.env ? import.meta.env.VITE_SUPABASE_ANON_KEY : "sb_publishable_E_QE1g8LMMI0h6zLZqyMTA_bCKzujHg";
const supabase = SUPABASE_URL && SUPABASE_ANON_KEY ? createClient(SUPABASE_URL, SUPABASE_ANON_KEY) : null;

const COLOR_POOL = [
  "bg-rose-100 text-rose-700 border-rose-200",
  "bg-blue-100 text-blue-700 border-blue-200",
  "bg-emerald-100 text-emerald-700 border-emerald-200",
  "bg-violet-100 text-violet-700 border-violet-200",
  "bg-amber-100 text-amber-700 border-amber-200",
  "bg-pink-100 text-pink-700 border-pink-200",
  "bg-cyan-100 text-cyan-700 border-cyan-200",
  "bg-orange-100 text-orange-700 border-orange-200",
];

function getUserStyle(index = 0) {
  return COLOR_POOL[Math.abs(Number(index) || 0) % COLOR_POOL.length];
}

function formatTime(iso) {
  if (!iso) return "";
  return new Intl.DateTimeFormat("de-DE", {
    hour: "2-digit",
    minute: "2-digit",
    day: "2-digit",
    month: "2-digit",
  }).format(new Date(iso));
}

function initials(name) {
  return (
    String(name || "?")
      .trim()
      .split(/\s+/)
      .filter(Boolean)
      .map((part) => part[0])
      .join("")
      .slice(0, 2)
      .toUpperCase() || "?"
  );
}

function renderRpgText(text) {
  const lines = String(text || "").split("\n");

  return lines.map((line, i) => {
    const parts = [];
    const regex = /(\*[^*]+\*)|("[^"]+")/g;
    let last = 0;
    let match;

    while ((match = regex.exec(line)) !== null) {
      if (match.index > last) {
        parts.push(line.slice(last, match.index));
      }

      const token = match[0];
      if (token.startsWith("*")) {
        parts.push(
          <strong key={`${i}-${match.index}`} className="font-semibold text-slate-100">
            {token.slice(1, -1)}
          </strong>
        );
      } else {
        parts.push(
          <em key={`${i}-${match.index}`} className="italic text-slate-200">
            {token}
          </em>
        );
      }

      last = regex.lastIndex;
    }

    if (last < line.length) {
      parts.push(line.slice(last));
    }

    return (
      <p key={i} className="whitespace-pre-wrap break-words leading-7">
        {parts.length ? parts : line}
      </p>
    );
  });
}

function runDevTests() {
  if (typeof window === "undefined") return;
  if (window.__RPG_DISCORD_TESTS__) return;

  window.__RPG_DISCORD_TESTS__ = true;

  console.assert(initials("Lara Mond") === "LM", "initials should return LM");
  console.assert(initials("") === "?", "initials should fallback to ?");
  console.assert(initials("single") === "S", "initials should work with one word");
  console.assert(getUserStyle(8) === COLOR_POOL[0], "style should wrap around");
  console.assert(getUserStyle(-1) === COLOR_POOL[1], "style should handle negative values safely");
  console.assert(Array.isArray(renderRpgText('*Test*')), "renderRpgText should return an array");
  console.assert(renderRpgText('*Test*').length === 1, "renderRpgText should return one line");
  console.assert(formatTime(null) === "", "formatTime should return empty string for empty input");
}

function AuthView({ email, password, setEmail, setPassword, onLogin, onSignup, loading }) {
  return (
    <div className="flex min-h-screen items-center justify-center bg-[#313338] p-6 text-white">
      <Card className="w-full max-w-md border-slate-700 bg-[#1e1f22] text-white shadow-2xl">
        <CardHeader>
          <CardTitle className="flex items-center gap-2 text-xl">
            <Sparkles className="h-5 w-5" />
            RPG Hub
          </CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          <div>
            <label className="mb-2 block text-sm text-slate-300">E-Mail</label>
            <Input
              className="border-slate-700 bg-[#111214] text-white"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              placeholder="name@email.de"
            />
          </div>
          <div>
            <label className="mb-2 block text-sm text-slate-300">Passwort</label>
            <Input
              className="border-slate-700 bg-[#111214] text-white"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="••••••••"
            />
          </div>
          <div className="grid grid-cols-2 gap-2">
            <Button onClick={onLogin} disabled={loading} className="bg-indigo-600 hover:bg-indigo-500">
              <LogIn className="mr-2 h-4 w-4" />
              Login
            </Button>
            <Button
              onClick={onSignup}
              disabled={loading}
              variant="outline"
              className="border-slate-700 bg-transparent text-white hover:bg-slate-800"
            >
              Registrieren
            </Button>
          </div>
        </CardContent>
      </Card>
    </div>
  );
}

function ProfileSetup({ username, setUsername, character, setCharacter, bio, setBio, onSave, loading }) {
  return (
    <div className="flex min-h-screen items-center justify-center bg-[#313338] p-6 text-white">
      <Card className="w-full max-w-lg border-slate-700 bg-[#1e1f22] text-white shadow-2xl">
        <CardHeader>
          <CardTitle>Profil erstellen</CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          <div>
            <label className="mb-2 block text-sm text-slate-300">Benutzername</label>
            <Input className="border-slate-700 bg-[#111214] text-white" value={username} onChange={(e) => setUsername(e.target.value)} />
          </div>
          <div>
            <label className="mb-2 block text-sm text-slate-300">Charaktername</label>
            <Input className="border-slate-700 bg-[#111214] text-white" value={character} onChange={(e) => setCharacter(e.target.value)} />
          </div>
          <div>
            <label className="mb-2 block text-sm text-slate-300">Kurzbeschreibung</label>
            <Textarea className="min-h-[100px] border-slate-700 bg-[#111214] text-white" value={bio} onChange={(e) => setBio(e.target.value)} />
          </div>
          <Button onClick={onSave} disabled={loading || !username.trim()} className="w-full bg-indigo-600 hover:bg-indigo-500">
            Profil speichern
          </Button>
        </CardContent>
      </Card>
    </div>
  );
}

function ChatMessage({ post, previousPost, profile, onReply }) {
  const grouped = previousPost?.user_id === post.user_id;

  return (
    <motion.div initial={{ opacity: 0, y: 8 }} animate={{ opacity: 1, y: 0 }} className={`group px-4 ${grouped ? "pt-1" : "pt-4"}`}>
      <div className="rounded-lg px-2 py-1 transition-colors hover:bg-white/5">
        <div className="flex gap-3">
          <div className="w-10 shrink-0">
            {!grouped && (
              <Avatar className="h-10 w-10 border border-slate-700">
                <AvatarFallback className={getUserStyle(profile?.color_index || 0)}>{initials(profile?.username || "?")}</AvatarFallback>
              </Avatar>
            )}
          </div>

          <div className="min-w-0 flex-1">
            {!grouped && (
              <div className="mb-1 flex flex-wrap items-center gap-2">
                <span className="font-semibold text-white">{profile?.username || "Unbekannt"}</span>
                <Badge variant="outline" className={`${getUserStyle(profile?.color_index || 0)} border`}>
                  {profile?.character_name || "Kein Charakter"}
                </Badge>
                <span className="text-xs text-slate-400">{formatTime(post.created_at)}</span>
              </div>
            )}

            <div className="text-sm text-slate-200">{renderRpgText(post.content)}</div>

            {post.parent_id && post.parent_preview && (
              <div className="mt-2 rounded-md border border-slate-700 bg-black/20 px-3 py-2 text-xs text-slate-400">
                Antwort auf: {post.parent_preview}
              </div>
            )}

            <button
              onClick={() => onReply(post)}
              className="mt-2 hidden items-center gap-1 text-xs text-indigo-300 hover:text-indigo-200 group-hover:inline-flex"
            >
              <Reply className="h-3.5 w-3.5" />
              Antworten
            </button>
          </div>
        </div>
      </div>
    </motion.div>
  );
}

export default function App() {
  const [session, setSession] = useState(null);
  const [profiles, setProfiles] = useState([]);
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(false);
  const [booting, setBooting] = useState(true);
  const [error, setError] = useState("");
  const [notice, setNotice] = useState("");

  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const [username, setUsername] = useState("");
  const [character, setCharacter] = useState("");
  const [bio, setBio] = useState("");

  const [composerText, setComposerText] = useState("");
  const [replyTo, setReplyTo] = useState(null);

  const bottomRef = useRef(null);

  const profilesMap = useMemo(() => new Map(profiles.map((p) => [p.id, p])), [profiles]);
  const ownProfile = profiles.find((p) => p.id === session?.user?.id) || null;

  const enrichedPosts = useMemo(() => {
    const map = new Map(posts.map((p) => [p.id, p]));
    return [...posts]
      .sort((a, b) => new Date(a.created_at) - new Date(b.created_at))
      .map((post) => ({
        ...post,
        parent_preview: post.parent_id ? map.get(post.parent_id)?.content?.slice(0, 100) || "Vorherige Nachricht" : null,
      }));
  }, [posts]);

  const loadData = useCallback(async () => {
    setBooting(true);
    setError("");

    const [{ data: profileData, error: profileError }, { data: postData, error: postError }] = await Promise.all([
      supabase.from("profiles").select("*").order("created_at", { ascending: true }),
      supabase.from("rpg_posts").select("*").order("created_at", { ascending: true }),
    ]);

    if (profileError) setError(profileError.message);
    if (postError) setError(postError.message);

    setProfiles(profileData || []);
    setPosts(postData || []);
    setBooting(false);
  }, []);

  useEffect(() => {
    runDevTests();
  }, []);

  const hasSupabaseConfig = Boolean(SUPABASE_URL && SUPABASE_ANON_KEY);

  useEffect(() => {
    if (!hasSupabaseConfig || !supabase) {
      setBooting(false);
      return;
    }

    let mounted = true;

    supabase.auth.getSession().then(async ({ data, error }) => {
      if (!mounted) return;
      if (error) setError(error.message);
      setSession(data.session ?? null);
      await loadData();
    });

    const { data: authListener } = supabase.auth.onAuthStateChange(async (_event, nextSession) => {
      if (!mounted) return;
      setSession(nextSession);
      setReplyTo(null);
      await loadData();
    });

    const postsChannel = supabase
      .channel("discord-posts")
      .on("postgres_changes", { event: "*", schema: "public", table: "rpg_posts" }, async () => {
        if (!mounted) return;
        const { data } = await supabase.from("rpg_posts").select("*").order("created_at", { ascending: true });
        if (mounted) setPosts(data || []);
      })
      .subscribe();

    const profilesChannel = supabase
      .channel("discord-profiles")
      .on("postgres_changes", { event: "*", schema: "public", table: "profiles" }, async () => {
        if (!mounted) return;
        const { data } = await supabase.from("profiles").select("*").order("created_at", { ascending: true });
        if (mounted) setProfiles(data || []);
      })
      .subscribe();

    return () => {
      mounted = false;
      authListener.subscription.unsubscribe();
      supabase.removeChannel(postsChannel);
      supabase.removeChannel(profilesChannel);
    };
  }, [hasSupabaseConfig, loadData]);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [enrichedPosts.length]);

  async function handleLogin() {
    if (!supabase) return;
    if (!email.trim() || !password.trim()) return;
    setLoading(true);
    setError("");

    const { error: loginError } = await supabase.auth.signInWithPassword({
      email: email.trim(),
      password: password.trim(),
    });

    if (loginError) setError(loginError.message);
    setLoading(false);
  }

  async function handleSignup() {
    if (!supabase) return;
    if (!email.trim() || !password.trim()) return;
    setLoading(true);
    setError("");
    setNotice("");

    const { data, error: signupError } = await supabase.auth.signUp({
      email: email.trim(),
      password: password.trim(),
    });

    if (signupError) setError(signupError.message);
    else if (data.user) setNotice("Registrierung erfolgreich. Je nach Einstellung musst du deine E-Mail noch bestätigen.");

    setLoading(false);
  }

  async function handleLogout() {
    if (!supabase) return;
    await supabase.auth.signOut();
    setReplyTo(null);
    setComposerText("");
  }

  async function createProfile() {
    if (!supabase) return;
    if (!session?.user?.id || !username.trim()) return;
    setLoading(true);
    setError("");
    setNotice("");

    const { error: profileError } = await supabase.from("profiles").upsert(
      {
        id: session.user.id,
        username: username.trim(),
        character_name: character.trim(),
        bio: bio.trim(),
        color_index: Math.floor(Math.random() * COLOR_POOL.length),
      },
      { onConflict: "id" }
    );

    if (profileError) {
      setError(profileError.message);
    } else {
      setNotice("Profil gespeichert.");
      await loadData();
    }

    setLoading(false);
  }

  async function sendPost() {
    if (!supabase) return;
    if (!session?.user?.id || !composerText.trim()) return;
    if (!ownProfile) {
      setError("Erstelle zuerst dein Profil.");
      return;
    }

    setLoading(true);
    setError("");
    setNotice("");

    const { error: postError } = await supabase.from("rpg_posts").insert({
      user_id: session.user.id,
      content: composerText.trim(),
      parent_id: replyTo?.id || null,
    });

    if (postError) {
      setError(postError.message);
    } else {
      setComposerText("");
      setReplyTo(null);
    }

    setLoading(false);
  }

  if (!hasSupabaseConfig) {
    return (
      <div className="flex min-h-screen items-center justify-center bg-[#313338] p-6 text-white">
        <Card className="w-full max-w-2xl border-slate-700 bg-[#1e1f22] text-white shadow-2xl">
          <CardHeader>
            <CardTitle>Deployment-Setup fehlt</CardTitle>
          </CardHeader>
          <CardContent className="space-y-4 text-sm text-slate-300">
            <p>Damit die Seite auf Vercel läuft, lege in deinem Projekt oder in Vercel unter Project Settings → Environment Variables diese zwei Werte an:</p>
            <div className="rounded-xl border border-slate-700 bg-[#111214] p-4 font-mono text-xs text-slate-200">
              <div>VITE_SUPABASE_URL=https://dein-projekt.supabase.co</div>
              <div>VITE_SUPABASE_ANON_KEY=dein_anon_key</div>
            </div>
            <p>Danach neu deployen.</p>
          </CardContent>
        </Card>
      </div>
    );
  }

  if (!session) {
    return (
      <AuthView
        email={email}
        password={password}
        setEmail={setEmail}
        setPassword={setPassword}
        onLogin={handleLogin}
        onSignup={handleSignup}
        loading={loading}
      />
    );
  }

  if (!ownProfile) {
    return (
      <ProfileSetup
        username={username}
        setUsername={setUsername}
        character={character}
        setCharacter={setCharacter}
        bio={bio}
        setBio={setBio}
        onSave={createProfile}
        loading={loading}
      />
    );
  }

  return (
    <div className="flex h-screen overflow-hidden bg-[#313338] text-white">
      <aside className="hidden w-[280px] flex-col border-r border-slate-800 bg-[#2b2d31] md:flex">
        <div className="border-b border-slate-800 px-4 py-4 shadow-sm">
          <div className="flex items-center gap-2 font-semibold">
            <MessageCircle className="h-4 w-4" />
            RPG Hub
          </div>
        </div>

        <div className="border-b border-slate-800 px-4 py-3">
          <div className="rounded-xl bg-[#1e1f22] p-3">
            <div className="flex items-center gap-3">
              <Avatar className="h-10 w-10 border border-slate-700">
                <AvatarFallback className={getUserStyle(ownProfile.color_index)}>{initials(ownProfile.username)}</AvatarFallback>
              </Avatar>
              <div className="min-w-0">
                <p className="truncate font-semibold">{ownProfile.username}</p>
                <p className="truncate text-xs text-slate-400">{ownProfile.character_name || "Kein Charakter"}</p>
              </div>
            </div>
          </div>
        </div>

        <div className="flex-1 overflow-hidden px-3 py-3">
          <div className="mb-3 flex items-center gap-2 px-1 text-xs uppercase tracking-wider text-slate-400">
            <Users className="h-3.5 w-3.5" />
            Spieler
          </div>
          <ScrollArea className="h-full pr-2">
            <div className="space-y-2">
              {profiles.map((profile) => (
                <div key={profile.id} className="flex items-center gap-3 rounded-xl px-3 py-2 hover:bg-white/5">
                  <Avatar className="h-9 w-9 border border-slate-700">
                    <AvatarFallback className={getUserStyle(profile.color_index)}>{initials(profile.username)}</AvatarFallback>
                  </Avatar>
                  <div className="min-w-0">
                    <p className="truncate text-sm font-medium text-slate-100">{profile.username}</p>
                    <p className="truncate text-xs text-slate-400">{profile.character_name || "Kein Charakter"}</p>
                  </div>
                </div>
              ))}
            </div>
          </ScrollArea>
        </div>

        <div className="flex gap-2 border-t border-slate-800 p-3">
          <Button variant="outline" className="flex-1 border-slate-700 bg-transparent text-white hover:bg-slate-800" onClick={loadData}>
            <RefreshCcw className="mr-2 h-4 w-4" />
            Reload
          </Button>
          <Button variant="outline" className="border-slate-700 bg-transparent text-white hover:bg-slate-800" onClick={handleLogout}>
            <LogOut className="h-4 w-4" />
          </Button>
        </div>
      </aside>

      <main className="flex min-w-0 flex-1 flex-col bg-[#313338]">
        <header className="border-b border-slate-800 bg-[#313338] px-4 py-3 shadow-sm">
          <div className="flex items-center justify-between gap-3">
            <div>
              <h1 className="text-lg font-semibold"># rpg-chat</h1>
              <p className="text-sm text-slate-400">Discord-artiger Live-Chat für dein schriftliches RPG</p>
            </div>
            <Dialog>
              <DialogTrigger asChild>
                <Button variant="outline" className="border-slate-700 bg-transparent text-white hover:bg-slate-800">
                  <Plus className="mr-2 h-4 w-4" />
                  SQL
                </Button>
              </DialogTrigger>
              <DialogContent className="max-w-3xl border-slate-700 bg-[#1e1f22] text-white">
                <DialogHeader>
                  <DialogTitle>Supabase SQL & Vercel Setup</DialogTitle>
                </DialogHeader>
                <ScrollArea className="max-h-[70vh] rounded-xl border border-slate-700 bg-[#111214] p-4">
                  <pre className="whitespace-pre-wrap text-xs text-slate-300">{`# .env
VITE_SUPABASE_URL=https://dein-projekt.supabase.co
VITE_SUPABASE_ANON_KEY=dein_anon_key

# SQL
create extension if not exists "pgcrypto";

create table if not exists public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  username text unique not null,
  character_name text,
  bio text,
  color_index int not null default 0,
  created_at timestamptz not null default now()
);

create table if not exists public.rpg_posts (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references public.profiles(id) on delete cascade,
  parent_id uuid null references public.rpg_posts(id) on delete cascade,
  content text not null,
  created_at timestamptz not null default now()
);

alter table public.profiles enable row level security;
alter table public.rpg_posts enable row level security;

create policy "profiles are readable by authenticated users"
on public.profiles
for select
to authenticated
using (true);

create policy "users can insert their own profile"
on public.profiles
for insert
to authenticated
with check (auth.uid() = id);

create policy "users can update their own profile"
on public.profiles
for update
to authenticated
using (auth.uid() = id)
with check (auth.uid() = id);

create policy "posts are readable by authenticated users"
on public.rpg_posts
for select
to authenticated
using (true);

create policy "users can insert their own posts"
on public.rpg_posts
for insert
to authenticated
with check (auth.uid() = user_id);

create policy "users can update their own posts"
on public.rpg_posts
for update
to authenticated
using (auth.uid() = user_id)
with check (auth.uid() = user_id);

create policy "users can delete their own posts"
on public.rpg_posts
for delete
to authenticated
using (auth.uid() = user_id);

alter publication supabase_realtime add table public.profiles;
alter publication supabase_realtime add table public.rpg_posts;

# Vercel Build Settings
Framework Preset: Vite
Build Command: npm run build
Output Directory: dist`}</pre>
                </ScrollArea>
              </DialogContent>
            </Dialog>
          </div>
        </header>

        {(error || notice || booting) && (
          <div className="px-4 pt-4">
            <Alert className={`${error ? "border-red-400 bg-red-500/10 text-red-100" : "border-emerald-400 bg-emerald-500/10 text-emerald-100"} border`}>
              <AlertDescription>{booting ? "Lade Nachrichten..." : error || notice}</AlertDescription>
            </Alert>
          </div>
        )}

        <div className="min-h-0 flex-1">
          <ScrollArea className="h-full">
            <div className="px-0 py-3">
              {enrichedPosts.length === 0 ? (
                <div className="px-6 py-10 text-center text-slate-400">Noch keine Nachrichten. Starte das RPG im Chat.</div>
              ) : (
                enrichedPosts.map((post, index) => (
                  <ChatMessage
                    key={post.id}
                    post={post}
                    previousPost={enrichedPosts[index - 1]}
                    profile={profilesMap.get(post.user_id)}
                    onReply={setReplyTo}
                  />
                ))
              )}
              <div ref={bottomRef} />
            </div>
          </ScrollArea>
        </div>

        <div className="border-t border-slate-800 bg-[#313338] p-4">
          {replyTo && (
            <div className="mb-3 flex items-center justify-between rounded-xl border border-slate-700 bg-[#1e1f22] px-3 py-2 text-sm text-slate-300">
              <div className="truncate">
                Antwort an <span className="font-semibold text-white">{profilesMap.get(replyTo.user_id)?.username || "Unbekannt"}</span>: {replyTo.content}
              </div>
              <Button variant="ghost" size="sm" className="text-slate-300 hover:bg-slate-800 hover:text-white" onClick={() => setReplyTo(null)}>
                Abbrechen
              </Button>
            </div>
          )}

          <div className="rounded-2xl border border-slate-700 bg-[#383a40] p-3">
            <Textarea
              value={composerText}
              onChange={(e) => setComposerText(e.target.value)}
              placeholder='Schreibe eine Nachricht... Beispiel: *Sie lehnt sich an den Türrahmen.* "Du bist spät."'
              className="min-h-[110px] resize-none border-0 bg-transparent px-0 text-white shadow-none placeholder:text-slate-400 focus-visible:ring-0"
              onKeyDown={(e) => {
                if (e.key === "Enter" && !e.shiftKey) {
                  e.preventDefault();
                  if (!loading && composerText.trim()) {
                    sendPost();
                  }
                }
              }}
            />
            <div className="mt-3 flex items-center justify-between gap-3">
              <p className="text-xs text-slate-400">Enter = senden · Shift + Enter = Zeilenumbruch · *Aktionen* fett · "Dialoge" kursiv</p>
              <Button onClick={sendPost} disabled={loading || !composerText.trim()} className="bg-indigo-600 hover:bg-indigo-500">
                Senden
              </Button>
            </div>
          </div>
        </div>
      </main>
    </div>
  );
}
