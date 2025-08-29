import React, { useMemo, useState, useEffect } from "react";
import { motion } from "framer-motion";
import {
  Card,
  CardContent,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Badge } from "@/components/ui/badge";
import { ChevronRight, Rocket, Shuffle, Save, Trash2, Copy, Download } from "lucide-react";

// --- Utility data ---
const GRADE_BANDS = [
  { id: "K-2", label: "K–2 (Primary)" },
  { id: "3-5", label: "3–5 (Upper Elem)" },
  { id: "6-8", label: "6–8 (Middle)" },
  { id: "9-12", label: "9–12 (High School)" },
];

const DOMAINS = [
  "Engineering Design",
  "Physical Science",
  "Life Science",
  "Earth/Space Science",
  "Computer Science / AI",
  "Math Integration",
];

const TIME_BOXES = ["30–45 min", "60–90 min", "2–3 class periods", "Multi‑week PBL"];

const SKILLS = [
  "Ask/Define", "Imagine", "Plan", "Create", "Test", "Improve",
  "Measure & Data", "Modeling", "Computational Thinking", "Collaboration", "Communication"
];

const CONSTRAINTS = [
  "Budget: $5 per team",
  "Only paper + tape allowed",
  "Must include a simple machine",
  "Must be biodegradable",
  "No electricity",
  "Must fit in a shoebox",
  "Survive a 1‑meter drop",
  "Travel at least 2 meters",
  "Use a sensor or switch",
  "Silent design (low noise)",
];

const MATERIALS_BANK = {
  common: [
    "paper", "cardboard", "index cards", "masking tape", "string", "rubber bands",
    "paper clips", "straws", "craft sticks", "plastic cups", "foil", "binder clips",
  ],
  k2: ["playdough", "pipe cleaners", "large beads", "cotton balls"],
  35: ["marbles", "clothespins", "balloons", "spoons"],
  68: ["syringes (no needles)", "tubing", "balsa wood", "magnets"],
  912: ["Arduino/micro:bit", "sensors", "3D‑printed parts", "PVC", "syringes & tubing"],
};

const CONTEXTS = [
  // community & environment
  "design a device that reduces waste at school",
  "build a model that slows runoff and prevents soil erosion",
  "create a solution to keep produce cool without electricity",
  "prototype a low‑cost water filter for turbid water",
  "engineer a pollinator‑friendly planter that resists heat",
  // physics & motion
  "launch a payload the farthest using only elastic energy",
  "design a vehicle that travels straight at least 2 meters",
  // life & health
  "prototype an assistive tool that increases grip strength",
  "design a prosthetic finger that can pick up small objects",
  // earth & space
  "build a quake‑resistant tower that protects a marshmallow",
  "design a wind device that lifts a paper clip payload",
  // CS & AI
  "create a paper machine that follows conditional rules (if/then)",
  "design an AI‑inspired classification game for sorting objects",
];

const SUCCESS_CRITERIA = [
  "meets the distance/height target",
  "uses no more than 10 items",
  "keeps cost under the budget",
  "runs for at least 10 seconds",
  "withstands a 1‑meter drop without damage",
  "filters 250 mL of water in under 3 minutes",
  "achieves at least 70% accuracy on a classification task",
  "moves at least 1 meter in a straight line",
  "supports a 300‑gram load for 10 seconds",
];

const RUBRIC_LEVELS = ["Beginning", "Developing", "Proficient", "Extending"];

function choice(arr) { return arr[Math.floor(Math.random() * arr.length)]; }
function shuffle(arr) { return [...arr].sort(() => Math.random() - 0.5); }

// --- Generator ---
function generateChallenge({ gradeBand, domain, timeBox, pickedSkills, customContext, customConstraints, customMaterials }) {
  const context = customContext?.trim() ? customContext.trim() : choice(CONTEXTS);
  const constraints = [choice(CONSTRAINTS), choice(CONSTRAINTS), ...(customConstraints?.trim() ? [customConstraints.trim()] : [])];

  const matBase = [
    ...MATERIALS_BANK.common,
    ...(gradeBand === "K-2" ? MATERIALS_BANK.k2 : []),
    ...(gradeBand === "3-5" ? MATERIALS_BANK[35] : []),
    ...(gradeBand === "6-8" ? MATERIALS_BANK[68] : []),
    ...(gradeBand === "9-12" ? MATERIALS_BANK[912] : []),
  ];
  const materialsList = shuffle(matBase).slice(0, 8).concat(customMaterials?.trim() ? customMaterials.split(",").map(s=>s.trim()).filter(Boolean) : []);

  const criteria = shuffle(SUCCESS_CRITERIA).slice(0, 2);

  const steps = [
    "Ask/Define: Re‑state the problem and list constraints & criteria.",
    "Imagine: Brainstorm at least 3 ideas with quick sketches.",
    "Plan: Choose one design and label materials & measurements.",
    "Create: Build a prototype within the time limit.",
    "Test: Collect data (distance/time/load/accuracy).",
    "Improve: Iterate based on results; document changes.",
  ];

  return {
    title: `${domain} Challenge for ${gradeBand}`,
    prompt: `Your team will ${context}. Follow the Engineering Design Process to build and test a prototype.`,
    gradeBand,
    domain,
    timeBox,
    skills: pickedSkills.length ? pickedSkills : [choice(SKILLS), choice(SKILLS)],
    constraints: Array.from(new Set(constraints)).slice(0, 3),
    materials: Array.from(new Set(materialsList)),
    criteria,
    steps,
  };
}

function RubricTable({ criteria }) {
  return (
    <div className="overflow-x-auto rounded-2xl border">
      <table className="w-full text-sm">
        <thead>
          <tr className="bg-muted/40">
            <th className="p-3 text-left">Dimension</th>
            {RUBRIC_LEVELS.map((lvl) => (
              <th key={lvl} className="p-3 text-left">{lvl}</th>
            ))}
          </tr>
        </thead>
        <tbody>
          {criteria.map((c, i) => (
            <tr key={i} className="border-t">
              <td className="p-3 font-medium">{c}</td>
              {RUBRIC_LEVELS.map((lvl, j) => (
                <td key={j} className="p-3 align-top">
                  {lvl === "Beginning" && "Limited evidence; design does not yet meet the criterion."}
                  {lvl === "Developing" && "Partial evidence; meets some aspects with support."}
                  {lvl === "Proficient" && "Meets the criterion with accurate data and rationale."}
                  {lvl === "Extending" && "Exceeds the criterion with optimization and clear justification."}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

function ChallengeCard({ data, onCopy, onSave }) {
  return (
    <Card className="shadow-lg">
      <CardHeader className="space-y-2">
        <div className="flex items-center gap-2">
          <Rocket className="h-5 w-5" />
          <CardTitle className="text-xl">{data.title}</CardTitle>
        </div>
        <div className="flex flex-wrap gap-2">
          <Badge variant="secondary">{data.gradeBand}</Badge>
          <Badge variant="secondary">{data.domain}</Badge>
          <Badge variant="secondary">Time: {data.timeBox}</Badge>
          {data.skills.map((s, i) => (<Badge key={i}>{s}</Badge>))}
        </div>
      </CardHeader>
      <CardContent className="space-y-4">
        <section>
          <h3 className="font-semibold mb-1">Challenge</h3>
          <p className="leading-6">{data.prompt}</p>
        </section>
        <section className="grid md:grid-cols-3 gap-4">
          <div>
            <h4 className="font-semibold">Constraints</h4>
            <ul className="list-disc pl-5">
              {data.constraints.map((c, i) => (<li key={i}>{c}</li>))}
            </ul>
          </div>
          <div>
            <h4 className="font-semibold">Success Criteria</h4>
            <ul className="list-disc pl-5">
              {data.criteria.map((c, i) => (<li key={i}>{c}</li>))}
            </ul>
          </div>
          <div>
            <h4 className="font-semibold">Suggested Materials</h4>
            <ul className="list-disc pl-5">
              {data.materials.map((m, i) => (<li key={i}>{m}</li>))}
            </ul>
          </div>
        </section>
        <section>
          <h4 className="font-semibold mb-1">Process Guide</h4>
          <ol className="list-decimal pl-5 space-y-1">
            {data.steps.map((s, i) => (<li key={i}>{s}</li>))}
          </ol>
        </section>
        <section className="space-y-2">
          <h4 className="font-semibold">Teacher Notes</h4>
          <ul className="list-disc pl-5 text-sm">
            <li>Safety first: preview materials and establish tool norms.</li>
            <li>Encourage multiple iterations and evidence‑based claims.</li>
            <li>Differentiation: adjust materials, time, and criteria for your learners.</li>
          </ul>
        </section>
        <section className="space-y-3">
          <h4 className="font-semibold">Rubric (editable after copy)</h4>
          <RubricTable criteria={data.criteria} />
        </section>
        <div className="flex gap-2 pt-2">
          <Button onClick={() => onCopy(data)} variant="secondary"><Copy className="h-4 w-4 mr-2"/>Copy</Button>
          <Button onClick={() => onSave(data)}><Save className="h-4 w-4 mr-2"/>Save</Button>
        </div>
      </CardContent>
    </Card>
  );
}

export default function STEMChallengeBuilder() {
  const [gradeBand, setGradeBand] = useState("3-5");
  const [domain, setDomain] = useState(DOMAINS[0]);
  const [timeBox, setTimeBox] = useState(TIME_BOXES[1]);
  const [pickedSkills, setPickedSkills] = useState(["Ask/Define", "Create"]);
  const [customContext, setCustomContext] = useState("");
  const [customConstraints, setCustomConstraints] = useState("");
  const [customMaterials, setCustomMaterials] = useState("");
  const [current, setCurrent] = useState(null);
  const [saved, setSaved] = useState([]);
  const [fileName, setFileName] = useState("stem-challenge");

  useEffect(() => {
    const s = localStorage.getItem("stem_challenges_v1");
    if (s) setSaved(JSON.parse(s));
  }, []);
  useEffect(() => {
    localStorage.setItem("stem_challenges_v1", JSON.stringify(saved));
  }, [saved]);

  const generated = useMemo(() => generateChallenge({
    gradeBand, domain, timeBox, pickedSkills, customContext, customConstraints, customMaterials
  }), [gradeBand, domain, timeBox, pickedSkills, customContext, customConstraints, customMaterials]);

  const handleShuffle = () => {
    setCurrent({ ...generateChallenge({ gradeBand, domain, timeBox, pickedSkills, customContext, customConstraints, customMaterials }) });
  };

  const handleCopy = async (data) => {
    const text = exportMarkdown(data);
    await navigator.clipboard.writeText(text);
  };

  const handleSave = (data) => {
    setSaved((prev) => [{ id: crypto.randomUUID(), createdAt: new Date().toISOString(), data }, ...prev]);
  };

  const handleDelete = (id) => setSaved((prev) => prev.filter((x) => x.id !== id));

  const toDownload = () => {
    const blob = new Blob([exportMarkdown(current || generated)], { type: "text/markdown" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = `${fileName || "stem-challenge"}.md`;
    a.click();
    URL.revokeObjectURL(url);
  };

  return (
    <div className="min-h-screen p-6 md:p-10 bg-gradient-to-b from-slate-50 to-white text-slate-900">
      <motion.div initial={{ opacity: 0, y: 10 }} animate={{ opacity: 1, y: 0 }} transition={{ duration: 0.4 }} className="mx-auto max-w-6xl space-y-6">
        <header className="flex items-center justify-between gap-4">
          <div>
            <h1 className="text-3xl font-bold tracking-tight">STEM Challenge Generator</h1>
            <p className="text-slate-600">Create ready‑to‑run challenges aligned to the Engineering Design Process.</p>
          </div>
          <div className="flex gap-2">
            <Input className="w-48" value={fileName} onChange={(e)=>setFileName(e.target.value)} placeholder="file name" />
            <Button onClick={toDownload}><Download className="h-4 w-4 mr-2"/>Export .md</Button>
          </div>
        </header>

        {/* Controls */}
        <Card>
          <CardContent className="p-4 md:p-6 grid gap-4 md:gap-6 md:grid-cols-4">
            <div className="space-y-2">
              <label className="text-sm font-medium">Grade Band</label>
              <div className="grid grid-cols-2 gap-2">
                {GRADE_BANDS.map((g) => (
                  <Button key={g.id} variant={gradeBand===g.id?"default":"secondary"} onClick={()=>setGradeBand(g.id)}>{g.label}</Button>
                ))}
              </div>
            </div>
            <div className="space-y-2 md:col-span-1">
              <label className="text-sm font-medium">Domain</label>
              <select className="w-full border rounded-xl p-2" value={domain} onChange={(e)=>setDomain(e.target.value)}>
                {DOMAINS.map((d)=> (<option key={d} value={d}>{d}</option>))}
              </select>
            </div>
            <div className="space-y-2 md:col-span-1">
              <label className="text-sm font-medium">Time</label>
              <select className="w-full border rounded-xl p-2" value={timeBox} onChange={(e)=>setTimeBox(e.target.value)}>
                {TIME_BOXES.map((t)=> (<option key={t} value={t}>{t}</option>))}
              </select>
            </div>
            <div className="space-y-2 md:col-span-1">
              <label className="text-sm font-medium">Focus Skills</label>
              <div className="flex flex-wrap gap-2">
                {SKILLS.map((s) => (
                  <button
                    key={s}
                    onClick={() => setPickedSkills((prev)=> prev.includes(s) ? prev.filter(x=>x!==s) : [...prev, s])}
                    className={`px-3 py-1 rounded-full border text-sm ${pickedSkills.includes(s)?"bg-slate-900 text-white":"bg-white"}`}
                  >{s}</button>
                ))}
              </div>
            </div>
            <div className="md:col-span-2 space-y-2">
              <label className="text-sm font-medium">Optional: Real‑world context</label>
              <Input value={customContext} onChange={(e)=>setCustomContext(e.target.value)} placeholder="e.g., design a cooling device for fruits at a food pantry" />
            </div>
            <div className="md:col-span-1 space-y-2">
              <label className="text-sm font-medium">Optional: Extra constraint</label>
              <Input value={customConstraints} onChange={(e)=>setCustomConstraints(e.target.value)} placeholder="e.g., must be rain‑proof" />
            </div>
            <div className="md:col-span-1 space-y-2">
              <label className="text-sm font-medium">Optional: Extra materials (comma‑sep)</label>
              <Input value={customMaterials} onChange={(e)=>setCustomMaterials(e.target.value)} placeholder="e.g., LEGO wheels, sand" />
            </div>
            <div className="md:col-span-4 flex flex-wrap items-center gap-3">
              <Button onClick={handleShuffle}><Shuffle className="h-4 w-4 mr-2"/>Generate</Button>
              <div className="text-sm text-slate-600 flex items-center gap-1"><ChevronRight className="h-4 w-4"/> Tip: Click a skill to toggle.</div>
            </div>
          </CardContent>
        </Card>

        {/* Output */}
        <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }}>
          <ChallengeCard data={current || generated} onCopy={handleCopy} onSave={handleSave} />
        </motion.div>

        {/* Saved library */}
        <section className="space-y-3">
          <h2 className="text-2xl font-semibold">Your Saved Challenges</h2>
          {saved.length === 0 && (
            <p className="text-slate-600">Nothing saved yet. Generate a challenge and click <span className="font-medium">Save</span>.</p>
          )}
          <div className="grid md:grid-cols-2 gap-4">
            {saved.map((item) => (
              <Card key={item.id}>
                <CardHeader className="pb-2">
                  <div className="flex items-center justify-between">
                    <CardTitle className="text-base">{item.data.title}</CardTitle>
                    <Button size="icon" variant="ghost" onClick={()=>handleDelete(item.id)}><Trash2 className="h-4 w-4"/></Button>
                  </div>
                  <div className="text-xs text-slate-500">Saved {new Date(item.createdAt).toLocaleString()}</div>
                </CardHeader>
                <CardContent className="space-y-2">
                  <div className="flex flex-wrap gap-2">
                    <Badge variant="secondary">{item.data.gradeBand}</Badge>
                    <Badge variant="secondary">{item.data.domain}</Badge>
                  </div>
                  <div className="flex gap-2">
                    <Button variant="secondary" onClick={()=>navigator.clipboard.writeText(exportMarkdown(item.data))}><Copy className="h-4 w-4 mr-2"/>Copy</Button>
                    <Button onClick={()=>setCurrent(item.data)}>Load</Button>
                  </div>
                </CardContent>
              </Card>
            ))}
          </div>
        </section>

        {/* About */}
        <footer className="pt-6 text-sm text-slate-500">
          <p>
            This tool assembles classroom‑ready STEM challenges using structured templates and randomized idea banks. 
            You can add your own contexts, constraints, and materials to tailor for your setting. Export as Markdown and paste into your LMS or a doc.
          </p>
        </footer>
      </motion.div>
    </div>
  );
}

// --- Export helpers ---
function exportMarkdown(d) {
  return `# ${d.title}\n\n` +
`**Grade Band:** ${d.gradeBand}  \n` +
`**Domain:** ${d.domain}  \n` +
`**Time:** ${d.timeBox}  \n` +
`**Focus Skills:** ${d.skills.join(", ")}\n\n` +
`## Challenge\n${d.prompt}\n\n` +
`## Constraints\n` + d.constraints.map((x)=>`- ${x}`).join("\n") + `\n\n` +
`## Success Criteria\n` + d.criteria.map((x)=>`- ${x}`).join("\n") + `\n\n` +
`## Suggested Materials\n` + d.materials.map((x)=>`- ${x}`).join("\n") + `\n\n` +
`## Process Guide\n` + d.steps.map((x,i)=>`${i+1}. ${x}`).join("\n") + `\n\n` +
`## Teacher Notes\n` +
`- Safety first: preview materials and establish tool norms.\n` +
`- Encourage multiple iterations and evidence‑based claims.\n` +
`- Differentiation: adjust materials, time, and criteria for your learners.\n`;
}

