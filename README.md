# To-do-list
import React, { useEffect, useState } from 'react';
import { motion } from 'framer-motion';
import { createClient } from '@supabase/supabase-js';
import { create } from 'zustand';
import { Chart as ChartJS, LineController, LineElement, PointElement, LinearScale, TimeScale, Title, Tooltip, Legend } from 'chart.js';
import 'chartjs-adapter-date-fns';

ChartJS.register(LineController, LineElement, PointElement, LinearScale, TimeScale, Title, Tooltip, Legend);

// ✅ Supabase Setup
const supabase = createClient(
  'https://syctjptdivhtlkgzlply.supabase.co',
  'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InN5Y3RqcHRkaXZodGxrZ3pscGx5Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTEyMDk1NTUsImV4cCI6MjA2Njc4NTU1NX0.ltNFj3dL2LjjyM4jIDA5Hhu_p9ps4WtIisTuzGyxlhE'
);

// ✅ Zustand Store
const useStore = create((set) => ({
  tasks: [],
  growth: 0,
  studyHours: [],
  weightData: [],
  learnings: [],
  isDarkMode: true,
  addTask: (task) => set((state) => ({
    tasks: [...state.tasks, { ...task, id: Date.now(), completed: false }]
  })),
  toggleTask: (id) => set((state) => ({
    tasks: state.tasks.map((task) =>
      task.id === id ? { ...task, completed: !task.completed } : task
    )
  })),
  updateGrowth: (value) => set((state) => ({
    growth: Math.min(state.growth + value, 100)
  })),
  addStudyHours: (hours) => set((state) => ({
    studyHours: [...state.studyHours, { date: new Date(), hours }]
  })),
  addWeight: (weight) => set((state) => ({
    weightData: [...state.weightData, { date: new Date(), weight }]
  })),
  addLearning: (text) => set((state) => ({
    learnings: [...state.learnings, { text, date: new Date() }]
  })),
  toggleDarkMode: () => set((state) => ({
    isDarkMode: !state.isDarkMode
  }))
}));

// ✅ Supabase Methods
const saveToSupabase = async (table, data) => {
  const { error } = await supabase.from(table).insert(data);
  if (error) console.error('Supabase error:', error);
};

const fetchFromSupabase = async (table) => {
  const { data, error } = await supabase.from(table).select();
  if (error) console.error('Supabase fetch error:', error);
  return data || [];
};

// ✅ TaskList
const TaskList = () => {
  const tasks = useStore((state) => state.tasks);
  const addTask = useStore((state) => state.addTask);
  const toggleTask = useStore((state) => state.toggleTask);
  const [taskText, setTaskText] = useState('');
  const [priority, setPriority] = useState('low');

  const handleAddTask = () => {
    if (taskText) {
      addTask({ text: taskText, priority });
      saveToSupabase('tasks', { text: taskText, priority, completed: false });
      setTaskText('');
    }
  };

  return (
    <motion.div initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }}
      className="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg">
      <h2 className="text-2xl font-bold mb-4 text-gray-800 dark:text-white">Daily Tasks</h2>
      <div className="flex mb-4">
        <input
          type="text"
          value={taskText}
          onChange={(e) => setTaskText(e.target.value)}
          placeholder="Add a new task..."
          className="flex-1 p-2 rounded-l-lg border dark:bg-gray-700 dark:text-white"
        />
        <select value={priority} onChange={(e) => setPriority(e.target.value)}
          className="p-2 border dark:bg-gray-700 dark:text-white">
          <option value="low">Low</option>
          <option value="medium">Medium</option>
          <option value="high">High</option>
        </select>
        <button onClick={handleAddTask}
          className="p-2 bg-blue-500 text-white rounded-r-lg hover:bg-blue-600">
          Add
        </button>
      </div>
      <ul>
        {tasks.map((task) => (
          <motion.li key={task.id}
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            className={`p-2 mb-2 rounded flex items-center ${
              task.priority === 'high' ? 'bg-red-100' :
              task.priority === 'medium' ? 'bg-yellow-100' : 'bg-green-100'
            } dark:bg-opacity-20`}>
            <input
              type="checkbox"
              checked={task.completed}
              onChange={() => toggleTask(task.id)}
              className="mr-2"
            />
            <span className={task.completed ? 'line-through' : ''}>{task.text}</span>
          </motion.li>
        ))}
      </ul>
    </motion.div>
  );
};

// ✅ DistractionBlocker
const DistractionBlocker = () => {
  const [time, setTime] = useState(25 * 60);
  const [isActive, setIsActive] = useState(false);
  const [isLocked, setIsLocked] = useState(false);

  useEffect(() => {
    let interval;
    if (isActive && time > 0) {
      interval = setInterval(() => setTime((t) => t - 1), 1000);
    } else if (time === 0) {
      setIsActive(false);
      setIsLocked(false);
    }
    return () => clearInterval(interval);
  }, [isActive, time]);

  const startTimer = () => {
    setIsActive(true);
    setIsLocked(true);
  };

  const resetTimer = () => {
    setIsActive(false);
    setTime(25 * 60);
    setIsLocked(false);
  };

  return (
    <motion.div initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }}
      className="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg">
      <h2 className="text-2xl font-bold mb-4 text-gray-800 dark:text-white">Distraction Blocker</h2>
      <div className="text-4xl font-mono mb-4">
        {Math.floor(time / 60)}:{(time % 60).toString().padStart(2, '0')}
      </div>
      <button onClick={startTimer} disabled={isActive}
        className="p-2 bg-blue-500 text-white rounded mr-2 hover:bg-blue-600 disabled:bg-gray-400">
        Start Pomodoro
      </button>
      <button onClick={resetTimer}
        className="p-2 bg-red-500 text-white rounded hover:bg-red-600">
        Reset
      </button>
      {isLocked && (
        <p className="mt-4 text-red-500">Focus Mode: Locked</p>
      )}
    </motion.div>
  );
};

// ✅ GrowthTracker
const GrowthTracker = () => {
  const growth = useStore((state) => state.growth);
  const updateGrowth = useStore((state) => state.updateGrowth);

  return (
    <motion.div initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }}
      className="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg">
      <h2 className="text-2xl font-bold mb-4 text-gray-800 dark:text-white">1% Growth Tracker</h2>
      <div className="w-full bg-gray-200 rounded-full h-4 dark:bg-gray-700">
        <motion.div className="bg-blue-500 h-4 rounded-full"
          style={{ width: `${growth}%` }}
          animate={{ width: `${growth}%` }}
          transition={{ duration: 0.5 }}
        />
      </div>
      <p className="mt-2 text-gray-600 dark:text-gray-300">{growth}% of daily growth achieved!</p>
      <button onClick={() => updateGrowth(10)}
        className="mt-4 p-2 bg-green-500 text-white rounded hover:bg-green-600">
        Add 10% Growth
      </button>
    </motion.div>
  );
};

// ✅ TimeTracking
const TimeTracking = () => {
  const studyHours = useStore((state) => state.studyHours);
  const weightData = useStore((state) => state.weightData);
  const addStudyHours = useStore((state) => state.addStudyHours);
  const addWeight = useStore((state) => state.addWeight);
  const [hours, setHours] = useState('');
  const [weight, setWeight] = useState('');

  useEffect(() => {
    const canvas = document.getElementById('studyChart');
    if (canvas) {
      const ctx = canvas.getContext('2d');
      const chart = new ChartJS(ctx, {
        type: 'line',
        data: {
          labels: studyHours.map((h) => new Date(h.date).toLocaleDateString()),
          datasets: [{
            label: 'Study Hours',
            data: studyHours.map((h) => h.hours),
            borderColor: 'blue',
            fill: false
          }]
        },
        options: {
          scales: {
            x: { type: 'time', time: { unit: 'day' } },
            y: { beginAtZero: true }
          }
        }
      });

      return () => chart.destroy(); // ✅ Fix: Clean up chart
    }
  }, [studyHours]);

  return (
    <motion.div initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }}
      className="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg">
      <h2 className="text-2xl font-bold mb-4 text-gray-800 dark:text-white">Time Tracking</h2>
      <div className="mb-4">
        <input type="number" value={hours} onChange={(e) => setHours(e.target.value)}
          placeholder="Study Hours" className="p-2 border rounded mr-2 dark:bg-gray-700 dark:text-white" />
        <button onClick={() => {
          if (hours) {
            addStudyHours(Number(hours));
            saveToSupabase('study_hours', { hours: Number(hours), date: new Date() });
            setHours('');
          }
        }} className="p-2 bg-blue-500 text-white rounded hover:bg-blue-600">
          Log Hours
        </button>
      </div>
      <div className="mb-4">
        <input type="number" value={weight} onChange={(e) => setWeight(e.target.value)}
          placeholder="Weight (kg)" className="p-2 border rounded mr-2 dark:bg-gray-700 dark:text-white" />
        <button onClick={() => {
          if (weight) {
            addWeight(Number(weight));
            saveToSupabase('weight_data', { weight: Number(weight), date: new Date() });
            setWeight('');
          }
        }} className="p-2 bg-blue-500 text-white rounded hover:bg-blue-600">
          Log Weight
        </button>
      </div>
      <canvas id="studyChart" className="w-full h-64"></canvas>
    </motion.div>
  );
};

// ✅ LearningLog
const LearningLog = () => {
  const learnings = useStore((state) => state.learnings);
  const addLearning = useStore((state) => state.addLearning);
  const [learningText, setLearningText] = useState('');

  return (
    <motion.div initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }}
      className="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg">
      <h2 className="text-2xl font-bold mb-4 text-gray-800 dark:text-white">What I Learned Today</h2>
      <textarea value={learningText} onChange={(e) => setLearningText(e.target.value)}
        placeholder="What did you learn today?" className="w-full p-2 border rounded mb-4 dark:bg-gray-700 dark:text-white" rows="4" />
      <button onClick={() => {
        if (learningText) {
          addLearning(learningText);
          saveToSupabase('learnings', { text: learningText, date: new Date() });
          setLearningText('');
        }
      }} className="p-2 bg-blue-500 text-white rounded hover:bg-blue-600">
        Save
      </button>
      <ul className="mt-4">
        {learnings.map((learning, index) => (
          <li key={index} className="p-2 border-b dark:border-gray-600">
            <span className="text-gray-600 dark:text-gray-300">
              {new Date(learning.date).toLocaleDateString()}:
            </span> {learning.text}
          </li>
        ))}
      </ul>
    </motion.div>
  );
};

// ✅ GoalCard
const GoalCard = () => {
  const [visible, setVisible] = useState(false);
  useEffect(() => {
    const interval = setInterval(() => {
      setVisible(true);
      setTimeout(() => setVisible(false), 5000);
    }, 30000);
    return () => clearInterval(interval);
  }, []);
  if (!visible) return null;
  return (
    <motion.div initial={{ opacity: 0, scale: 0.5 }} animate={{ opacity: 1, scale: 1 }}
      exit={{ opacity: 0, scale: 0.5 }}
      className="fixed bottom-4 right-4 bg-blue-500 text-white p-4 rounded-lg shadow-lg">
      <h3 className="font-bold">Goal Reminder</h3>
      <p>Keep pushing towards your dreams!</p>
    </motion.div>
  );
};

// ✅ App Root
const App = () => {
  const isDarkMode = useStore((state) => state.isDarkMode);
  const toggleDarkMode = useStore((state) => state.toggleDarkMode);

  useEffect(() => {
    const loadData = async () => {
      const tasks = await fetchFromSupabase('tasks');
      const studyHours = await fetchFromSupabase('study_hours');
      const weightData = await fetchFromSupabase('weight_data');
      const learnings = await fetchFromSupabase('learnings');
      useStore.setState({ tasks, studyHours, weightData, learnings });
    };
    loadData();
  }, []);

  return (
    <div className={`${isDarkMode ? 'dark' : ''} min-h-screen bg-gray-100 dark:bg-gray-900 transition-colors duration-300`}>
      <div className="container mx-auto p-4">
        <div className="flex justify-between items-center mb-6">
          <h1 className="text-3xl font-bold text-gray-800 dark:text-white">Growth Dashboard</h1>
          <button onClick={toggleDarkMode}
            className="p-2 bg-gray-200 dark:bg-gray-700 rounded">
            {isDarkMode ? '☀️ Light' : '🌙 Dark'}
          </button>
        </div>
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          <TaskList />
          <DistractionBlocker />
          <GrowthTracker />
          <TimeTracking />
          <LearningLog />
        </div>
        <GoalCard />
      </div>
    </div>
  );
};

export default App;
