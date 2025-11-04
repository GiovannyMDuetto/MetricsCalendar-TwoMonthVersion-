# MetricsCalendar-TwoMonthVersion-

```
import { useState } from "react";
import { defineProperties } from "figma:react";
import { motion } from "motion/react";

// SVG paths used in the component
const svgPaths = {
  p10001380: "M7.41 1.41L6 0L0 6L6 12L7.41 10.59L2.83 6L7.41 1.41Z",
  p25284240: "M1.41 0L0 1.41L4.58 6L0 10.59L1.41 12L7.41 6L1.41 0Z",
  p2d18c680: "M8 0L6.59 1.41L11.17 6L6.59 10.59L8 12L14 6L8 0Z",
  p67fcb00: "M6 12L7.41 10.59L2.83 6L7.41 1.41L6 0L0 6L6 12Z",
  p8039dc0: "M12.59 12L14 10.59L9.42 6L14 1.41L12.59 0L6.59 6L12.59 12Z",
};

// Month names array for date formatting
const MONTHS = [
  "January", "February", "March", "April", "May", "June", 
  "July", "August", "September", "October", "November", "December"
];

// One-time generated percentage values for days 1-31 (stable throughout the session)
const PERCENTAGES = Array.from({ length: 31 }, () => Math.floor(Math.random() * 101));

// Data structure for indicators
// For each day (1-31), specifies which indicators to show:
// - hasEvent: blue rectangle
// - hasNote: purple diamond
// - hasRestriction: orange circle
const DAY_INDICATORS = Array.from({ length: 31 }, (_, i) => {
  // Create a variety of indicator patterns
  const patterns = [
    { hasEvent: true, hasNote: true, hasRestriction: true }, // All three
    { hasEvent: true, hasNote: false, hasRestriction: false }, // Just event
    { hasEvent: false, hasNote: true, hasRestriction: false }, // Just note
    { hasEvent: false, hasNote: false, hasRestriction: true }, // Just restriction
    { hasEvent: true, hasNote: true, hasRestriction: false }, // Event + Note
    { hasEvent: true, hasNote: false, hasRestriction: true }, // Event + Restriction
    { hasEvent: false, hasNote: true, hasRestriction: true }, // Note + Restriction
    { hasEvent: false, hasNote: false, hasRestriction: false }, // None
  ];
  
  // Distribute patterns somewhat evenly
  return patterns[i % patterns.length];
});

// Days with anomalies - these will have the teal pulsing background
// Strategically chosen days for visual impact (including weekends and mid-week)
const ANOMALY_DAYS = [3, 7, 14, 21, 28];

// Metric dropdown options
const METRIC_OPTIONS = [
  "ADR (Commit)",
  "Occupancy (Commit)",
  "Occupancy (Duetto Forecast)",
  "Occupancy (Locked Forecast)",
  "Room Revenue (Commit)",
  "Room Revenue (Duetto Forecast)",
  "Room Revenue (Locked Forecast)",
  "RevPar (Commit)",
  "Competitor Average",
  "My Rate (Shopped)",
  "My Rate (Bar)",
  "Recommended Rate",
  "Occupancy (Demand)"
] as const;

// Format value based on selected metric
function formatMetricValue(percentage, metricType) {
  // Return dash for metrics that should show "-"
  if (
    metricType === "Competitor Average" ||
    metricType === "My Rate (Shopped)" ||
    metricType === "Recommended Rate"
  ) {
    return "-";
  }
  
  // Return percentage for Occupancy metrics
  if (
    metricType === "Occupancy (Commit)" ||
    metricType === "Occupancy (Duetto Forecast)" ||
    metricType === "Occupancy (Locked Forecast)" ||
    metricType === "Occupancy (Demand)"
  ) {
    return `${percentage}%`;
  }
  
  // Return $XXK format for Room Revenue metrics
  if (
    metricType === "Room Revenue (Commit)" ||
    metricType === "Room Revenue (Duetto Forecast)" ||
    metricType === "Room Revenue (Locked Forecast)"
  ) {
    // Convert percentage to a value in thousands
    const value = Math.round((percentage / 100) * 50); // Scale percentage to a reasonable value
    return `$${value}K`;
  }
  
  // Return $XXX format for ADR, RevPar and My Rate (Bar)
  if (
    metricType === "ADR (Commit)" ||
    metricType === "RevPar (Commit)" ||
    metricType === "My Rate (Bar)"
  ) {
    // Convert percentage to a dollar amount
    const value = Math.round((percentage / 100) * 300) + 99; // Scale to a reasonable hotel rate
    return `$${value}`;
  }
  
  // Default fallback to percentage
  return `${percentage}%`;
}

// Navigation button component
function DatePickerMonthNavButton({ 
  icon = null, 
  hoverState = "false", 
  onClick, 
  className = "" 
}) {
  if (hoverState === "true") {
    return (
      <div 
        className={`bg-neutral-100 box-border content-stretch flex flex-row items-center justify-start p-0 relative rounded-[32px] size-full cursor-pointer ${className}`}
        onClick={onClick}
      >
        <div className="overflow-clip relative shrink-0 size-6 rounded-[32px]">
          <div className="absolute bottom-1/4 left-[34.563%] right-[34.563%] top-1/4">
            <svg className="block size-full" fill="none" preserveAspectRatio="none" role="presentation" viewBox="0 0 8 12">
              <g id="Vector">
                <path d={svgPaths.p10001380} fill="#585858" />
              </g>
            </svg>
          </div>
        </div>
      </div>
    );
  }
  
  return (
    <div 
      className={`box-border content-stretch flex flex-row items-center justify-start p-0 relative rounded-[32px] size-full cursor-pointer ${className}`}
      onClick={onClick}
    >
      {icon || (
        <div className="overflow-clip relative shrink-0 size-6 rounded-[32px]">
          <div className="absolute bottom-1/4 left-[34.563%] right-[34.563%] top-1/4">
            <svg className="block size-full" fill="none" preserveAspectRatio="none" role="presentation" viewBox="0 0 8 12">
              <g id="Vector">
                <path d={svgPaths.p10001380} fill="#585858" />
              </g>
            </svg>
          </div>
        </div>
      )}
    </div>
  );
}

// Date shortcuts component
function DateShortcuts({ onSelectRange, currentStartDate, currentEndDate }) {
  // Helper to create date ranges
  const createDateRange = (start, end) => {
    return {
      startDate: {
        day: start.getDate(),
        month: start.getMonth(),
        year: start.getFullYear()
      },
      endDate: {
        day: end.getDate(),
        month: end.getMonth(),
        year: end.getFullYear()
      }
    };
  };

  // Date range generators
  const getYesterday = () => {
    const yesterday = new Date();
    yesterday.setDate(yesterday.getDate() - 1);
    return createDateRange(yesterday, yesterday);
  };

  const getToday = () => {
    const today = new Date();
    return createDateRange(today, today);
  };

  const getTomorrow = () => {
    const tomorrow = new Date();
    tomorrow.setDate(tomorrow.getDate() + 1);
    return createDateRange(tomorrow, tomorrow);
  };

  const getPreviousWeek = () => {
    const today = new Date();
    const start = new Date();
    start.setDate(today.getDate() - 7);
    const end = new Date(today);
    end.setDate(today.getDate() - 1);
    return createDateRange(start, end);
  };

  const getPreviousMonth = () => {
    const today = new Date();
    const start = new Date(today.getFullYear(), today.getMonth() - 1, 1);
    const end = new Date(today.getFullYear(), today.getMonth(), 0);
    return createDateRange(start, end);
  };

  const getPreviousQuarter = () => {
    const today = new Date();
    const currentQuarter = Math.floor(today.getMonth() / 3);
    const start = new Date(today.getFullYear(), (currentQuarter - 1) * 3, 1);
    const end = new Date(today.getFullYear(), currentQuarter * 3, 0);
    return createDateRange(start, end);
  };

  const getCurrentWeek = () => {
    const today = new Date();
    const start = new Date();
    // Get the start of the current week (Sunday)
    start.setDate(today.getDate() - today.getDay());
    const end = new Date(start);
    end.setDate(start.getDate() + 6);
    return createDateRange(start, end);
  };

  const getCurrentMonth = () => {
    const today = new Date();
    const start = new Date(today.getFullYear(), today.getMonth(), 1);
    const end = new Date(today.getFullYear(), today.getMonth() + 1, 0);
    return createDateRange(start, end);
  };

  const getNext7Days = () => {
    const today = new Date();
    const end = new Date();
    end.setDate(today.getDate() + 6);
    return createDateRange(today, end);
  };

  const getNext14Days = () => {
    const today = new Date();
    const end = new Date();
    end.setDate(today.getDate() + 13);
    return createDateRange(today, end);
  };

  const getNext30Days = () => {
    const today = new Date();
    const end = new Date();
    end.setDate(today.getDate() + 29);
    return createDateRange(today, end);
  };

  const getNext90Days = () => {
    const today = new Date();
    const end = new Date();
    end.setDate(today.getDate() + 89);
    return createDateRange(today, end);
  };

  // Check if two date ranges are equal
  const isRangeEqual = (range) => {
    if (!currentStartDate || !currentEndDate) return false;
    
    return (
      currentStartDate.day === range.startDate.day &&
      currentStartDate.month === range.startDate.month &&
      currentStartDate.year === range.startDate.year &&
      currentEndDate.day === range.endDate.day &&
      currentEndDate.month === range.endDate.month &&
      currentEndDate.year === range.endDate.year
    );
  };

  // Shortcut option component
  const ShortcutOption = ({ label, getRange }) => {
    const range = getRange();
    const selected = isRangeEqual(range);
    
    return (
      <button
        className={`w-full text-left py-1.5 px-3 ml-2 text-[14px] font-lato rounded-[40px] transition-colors ${
          selected 
            ? "bg-[#1B4DC0] text-white" 
            : "text-[#1b4dc0] hover:bg-gray-100"
        }`}
        onClick={() => onSelectRange(range)}
      >
        {label}
      </button>
    );
  };

  // Section title component
  const SectionTitle = ({ title }) => {
    return (
      <div className="text-[#585858] font-lato text-[15px] font-bold mt-3 mb-1">
        {title}
      </div>
    );
  };

  return (
    <div className="box-border content-stretch flex flex-col items-start justify-start p-4 relative shrink-0 w-[200px] border-l border-[#e0e0e0]">
      {/* Top options */}
      <ShortcutOption label="Yesterday" getRange={getYesterday} />
      <ShortcutOption label="Today" getRange={getToday} />
      <ShortcutOption label="Tomorrow" getRange={getTomorrow} />
      
      {/* Divider */}
      <div className="my-3 w-full h-px bg-[#e0e0e0]" />
      
      {/* Previous section */}
      <SectionTitle title="Previous" />
      <ShortcutOption label="Week" getRange={getPreviousWeek} />
      <ShortcutOption label="Month" getRange={getPreviousMonth} />
      <ShortcutOption label="Quarter" getRange={getPreviousQuarter} />
      
      {/* Current section */}
      <SectionTitle title="Current" />
      <ShortcutOption label="Week" getRange={getCurrentWeek} />
      <ShortcutOption label="Month" getRange={getCurrentMonth} />
      
      {/* Next section */}
      <SectionTitle title="Next" />
      <ShortcutOption label="7 days" getRange={getNext7Days} />
      <ShortcutOption label="14 days" getRange={getNext14Days} />
      <ShortcutOption label="30 days" getRange={getNext30Days} />
      <ShortcutOption label="90 days" getRange={getNext90Days} />
    </div>
  );
}

// Metric selector component
function MetricSelector({ selected, onChange }) {
  return (
    <div className="self-end w-[392px]">
      <select
        value={selected}
        onChange={(e) => onChange(e.target.value)}
        className="w-full px-4 py-2 rounded-md bg-white border border-[#E0E0E0] text-[#585858] text-sm outline-none focus:ring-2 focus:ring-[#2D64E8]"
      >
        {METRIC_OPTIONS.map((opt) => (
          <option key={opt} value={opt} className="text-[#585858]">
            {opt}
          </option>
        ))}
      </select>
    </div>
  );
}

// Menu divider component
function MenuDivider() {
  return (
    <div className="box-border content-stretch flex flex-col gap-2 items-start justify-start p-0 relative size-full">
      <div className="bg-[#e0e0e0] h-px shrink-0 w-full" />
    </div>
  );
}

// Spacer component
function Spacer() {
  return <div className="shrink-0 size-6" />;
}

// Empty calendar day cell
function EmptyCell() {
  return <div className="size-14 relative shrink-0" />;
}

// Week day legend component
function DatePickerWeekLegend() {
  return (
    <div className="box-border content-stretch flex flex-row font-lato items-center justify-start leading-[0] not-italic px-0 py-2 relative shrink-0 text-[#585858] text-[16px] text-center w-full">
      {["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"].map((day, index) => (
        <div key={index} className="flex flex-col justify-center relative shrink-0 w-14">
          <p className="block leading-[normal]">{day}</p>
        </div>
      ))}
    </div>
  );
}

// Filter button component
function FilterButton({ active, onClick, label, children }) {
  return (
    <button
      className={`flex items-center gap-2 px-4 py-2 rounded-md min-w-[110px] ${
        active ? 'bg-[#F5F5F5]' : 'bg-[#FFFFFF]'
      } transition-colors duration-200 ease-in-out`}
      onClick={onClick}
    >
      <span className="flex items-center justify-center w-5 h-5">
        {children}
      </span>
      <span className="text-[#585858] text-sm font-lato">{label}</span>
    </button>
  );
}

// Indicator filter bar component
function IndicatorFilterBar({
  showEvents,
  setShowEvents,
  showNotes,
  setShowNotes,
  showRestrictions,
  setShowRestrictions,
  showAnomalies,
  setShowAnomalies,
}) {
  return (
    <div className="grid grid-cols-2 gap-2 py-3 self-end justify-end w-[392px]">
      {/* Row 1 */}
      <FilterButton 
        active={showEvents}
        onClick={() => setShowEvents(!showEvents)}
        label="Events"
      >
        <div className="w-[6px] h-[6px] bg-[#2D64E8] rounded-full" />
      </FilterButton>

      <FilterButton 
        active={showNotes}
        onClick={() => setShowNotes(!showNotes)}
        label="Notes"
      >
        <div className="w-[8.49px] h-[8.49px] bg-[#AF75D9] transform rotate-45" />
      </FilterButton>

      {/* Row 2 */}
      <FilterButton 
        active={showRestrictions}
        onClick={() => setShowRestrictions(!showRestrictions)}
        label="Restrictions"
      >
        <div className="w-[6px] h-[6px] bg-[#FF9800] rounded-full" />
      </FilterButton>

      <FilterButton
        active={showAnomalies}
        onClick={() => setShowAnomalies(!showAnomalies)}
        label="Anomalies"
      >
        {/* Pulsing teal dot */}
        <motion.div
          className="w-[6px] h-[6px] rounded-full bg-[#00ACC1]"
          animate={{ opacity: [0.4, 1, 0.4], scale: [0.9, 1.2, 0.9] }}
          transition={{ duration: 1, repeat: Infinity, ease: 'easeInOut' }}
        />
      </FilterButton>
    </div>
  );
}

// Day cell component
function DatePickerDay({
  day,
  month,
  onClick,
  isSelected,
  isRangeStart,
  isRangeEnd,
  isInRange,
  onHover,
  hoverDate,
  startDate,
  endDate,
  isHovered,
  showIndicators = true,
  percentage,
  filterSettings,
  selectedMetric
}) {
  const { showEvents, showNotes, showRestrictions, showAnomalies } = filterSettings || {
    showEvents: true,
    showNotes: true,
    showRestrictions: true,
    showAnomalies: true,
  };
  
  // Determine if this day is an anomaly
  const isAnomaly = ANOMALY_DAYS.includes(day);
  
  // Text class based on anomaly status (update to darker teal & bold when anomaly)
  const textClass = isAnomaly 
    ? "text-[#00838F] font-bold" 
    : "text-[#585858]";
  
  // Get indicators for this day
  const dayIndicators = DAY_INDICATORS[day - 1] || { hasEvent: false, hasNote: false, hasRestriction: false };
  const { hasEvent, hasNote, hasRestriction } = dayIndicators;
  const hasAnyIndicator = hasEvent || hasNote || hasRestriction;
  
  // Determine if this day is the start/end of hover selection
  const isHoverSelectionStart = startDate && !endDate && hoverDate && (
    (startDate.day === day && startDate.month === month && 
      ((startDate.month === hoverDate.month && startDate.day > hoverDate.day) || 
       (startDate.month > hoverDate.month))) || 
    (hoverDate.day === day && hoverDate.month === month && 
      ((startDate.month === hoverDate.month && startDate.day > hoverDate.day) || 
       (startDate.month > hoverDate.month)))
  );
  
  const isHoverSelectionEnd = startDate && !endDate && hoverDate && (
    (startDate.day === day && startDate.month === month && 
      ((startDate.month === hoverDate.month && startDate.day < hoverDate.day) || 
       (startDate.month < hoverDate.month))) || 
    (hoverDate.day === day && hoverDate.month === month && 
      ((startDate.month === hoverDate.month && startDate.day < hoverDate.day) || 
       (startDate.month < hoverDate.month)))
  );
  
  const isHoverSelectionMiddle = startDate && !endDate && hoverDate && 
    !isHoverSelectionStart && !isHoverSelectionEnd && (
      (startDate.month === hoverDate.month && month === startDate.month && 
        day > Math.min(startDate.day, hoverDate.day) && 
        day < Math.max(startDate.day, hoverDate.day)) || 
      (startDate.month !== hoverDate.month && (
        (month === startDate.month && startDate.month < hoverDate.month && day > startDate.day) || 
        (month === startDate.month && startDate.month > hoverDate.month && day < startDate.day) || 
        (month === hoverDate.month && startDate.month < hoverDate.month && day < hoverDate.day) || 
        (month === hoverDate.month && startDate.month > hoverDate.month && day > hoverDate.day) || 
        (month > startDate.month && month < hoverDate.month) || 
        (month < startDate.month && month > hoverDate.month)
      ))
    );

  // Format value based on selected metric
  const formattedValue = formatMetricValue(percentage, selectedMetric);

  return (
    <div className="flex flex-col items-center w-14">
      <div
        className="relative w-full aspect-square cursor-pointer"
        onClick={onClick}
        onMouseEnter={onHover}
      >
        {/* Anomaly pulsing background (smaller, above selection) */}
        {isAnomaly && showAnomalies && (
          <motion.div 
            className="absolute inset-1 z-10 bg-[#CAF7FB] rounded-[28px]"
            animate={{ 
              opacity: [0.4, 0.9, 0.4],
              scale: [0.95, 1.05, 0.95]
            }}
            transition={{
              duration: 1.2,
              ease: "easeInOut",
              repeat: Infinity,
              repeatType: "loop"
            }}
          />
        )}
        
        {/* Selection background - always rendered beneath other overlays */}
        {(isSelected || isRangeStart || isRangeEnd || isInRange || isHovered || 
          isHoverSelectionStart || isHoverSelectionEnd || isHoverSelectionMiddle) && (
          <div 
            className={`absolute inset-0 z-0 ${
              // Single day selection or hover - fully rounded
              (isSelected && !isRangeStart && !isRangeEnd) || 
              (isHovered && !startDate) || 
              (startDate && !endDate && isHovered && startDate.day === day && startDate.month === month)
                ? "rounded-[32px] bg-[#E8EFFF]"
              // Range start or hover selection start - left rounded only
              : (isRangeStart || isHoverSelectionStart)
                ? "rounded-l-[32px] bg-[#E8EFFF]"
              // Range end or hover selection end - right rounded only
              : (isRangeEnd || isHoverSelectionEnd)
                ? "rounded-r-[32px] bg-[#E8EFFF]"
              // Middle days - no rounding
              : "bg-[#E8EFFF]"
            }`}
          />
        )}
        
        {/* Black overlay for selected range endpoints */}
        {(isRangeStart || isRangeEnd || isHoverSelectionStart || isHoverSelectionEnd) && (
          <div className="absolute inset-0 z-10 bg-black opacity-10 rounded-[32px]" />
        )}
        
        {/* Content container - for centered alignment */}
        <div className="absolute inset-0 flex flex-col items-center justify-center">
          {/* Date text - always on top */}
          <div className={`z-20 flex flex-col font-lato justify-center leading-[0] not-italic ${isAnomaly && showAnomalies ? 'text-[#00838F] font-bold' : 'text-[#585858]'} text-[16px] text-center`}>
            <p className="block leading-[normal]">{day}</p>
          </div>
          
          {/* Indicators container - always present for consistent spacing */}
          <div className="h-[6px] flex flex-row items-center justify-center gap-[3px] mt-1 z-20">
            {/* Selectively render indicators based on filter settings */}
            {showIndicators && hasEvent && showEvents && (
              <div className="w-[6px] h-[6px] bg-[#2D64E8] rounded-full" />
            )}
            {showIndicators && hasNote && showNotes && (
              <div className="w-[8.49px] h-[8.49px] bg-[#AF75D9] transform rotate-45" />
            )}
            {showIndicators && hasRestriction && showRestrictions && (
              <div className="w-[6px] h-[6px] bg-[#FF9800] rounded-full" />
            )}
          </div>
        </div>
      </div>
      {/* Formatted metric value outside the clickable area */}
      <p className="text-[#757575] text-[12px] leading-none mt-1">{formattedValue}</p>
    </div>
  );
}

// Actions bar component with Apply and Cancel buttons
function ActionsBar({ onApply, onCancel }) {
  return (
    <div className="box-border content-stretch flex flex-row items-center justify-end gap-2 pb-0 pt-4 px-0 relative shrink-0 w-full">
      <div className="absolute border-[#e0e0e0] border-t border-solid inset-0 pointer-events-none" />
      <div 
        className="box-border content-stretch flex flex-row gap-2 h-9 items-center justify-center px-4 py-2 relative rounded shrink-0 cursor-pointer"
        onClick={onCancel}
      >
        <div className="flex flex-col font-lato justify-center leading-[0] not-italic relative shrink-0 text-[#1b4dc0] text-[14px] text-left text-nowrap">
          <p className="block leading-[normal] whitespace-pre">Cancel</p>
        </div>
      </div>
      <div 
        className="bg-[#1b4dc0] box-border content-stretch flex flex-row gap-2 h-9 items-center justify-center px-4 py-2 relative rounded shrink-0 cursor-pointer"
        onClick={onApply}
      >
        <div className="flex flex-col font-lato justify-center leading-[0] not-italic relative shrink-0 text-[#ffffff] text-[14px] text-left text-nowrap">
          <p className="block leading-[normal] whitespace-pre">Apply</p>
        </div>
      </div>
    </div>
  );
}

// Function to generate calendar data for a specific month and year
function generateCalendarData(month, year) {
  const firstDay = new Date(year, month, 1).getDay(); // 0 = Sunday, 1 = Monday, etc.
  const daysInMonth = new Date(year, month + 1, 0).getDate();
  
  let calendar = [];
  
  // Create the first week with placeholders for days before the 1st
  let firstWeek = Array(7).fill(null);
  for (let i = 0; i < 7; i++) {
    if (i >= firstDay) {
      firstWeek[i] = i - firstDay + 1;
    }
  }
  calendar.push(firstWeek);
  
  // Fill in the rest of the weeks
  let day = 8 - firstDay; // Start with the day after the last day in the first week
  while (day <= daysInMonth) {
    let week = Array(7).fill(null);
    for (let i = 0; i < 7 && day <= daysInMonth; i++) {
      week[i] = day++;
    }
    calendar.push(week);
  }
  
  return calendar;
}

// Month component
function Month({ 
  displayMonth, 
  displayYear, 
  startDate, 
  endDate, 
  onDayClick, 
  hoverDate, 
  onDayHover, 
  onNavigateMonth,
  filterSettings,
  selectedMetric,
  hideLeftNav = false,
  hideRightNav = false,
}) {
  const monthKey = `${displayMonth}-${displayYear}`;
  const monthTitle = `${MONTHS[displayMonth]} ${displayYear}`;
  
  // Generate calendar data for this month
  const calendarData = generateCalendarData(displayMonth, displayYear);
  
  // Determine if a day is in the selected range
  const isDayInRange = (day, monthKey) => {
    if (!startDate || !endDate || !day) return false;
    
    const currentMonthYear = monthKey.split("-");
    const currentMonth = parseInt(currentMonthYear[0]);
    const currentYear = parseInt(currentMonthYear[1]);
    
    const currentDate = new Date(currentYear, currentMonth, day);
    const start = new Date(startDate.year, startDate.month, startDate.day);
    const end = new Date(endDate.year, endDate.month, endDate.day);
    
    return currentDate >= start && currentDate <= end;
  };
  
  // Determine if day is in the hover range (when selecting)
  const isDayInHoverRange = (day, monthKey) => {
    if (!startDate || !hoverDate || endDate || !day) return false;
    
    const currentMonthYear = monthKey.split("-");
    const currentMonth = parseInt(currentMonthYear[0]);
    const currentYear = parseInt(currentMonthYear[1]);
    
    const currentDate = new Date(currentYear, currentMonth, day);
    const start = new Date(startDate.year, startDate.month, startDate.day);
    const hover = new Date(hoverDate.year, hoverDate.month, hoverDate.day);
    
    return (currentDate >= start && currentDate <= hover) || 
           (currentDate >= hover && currentDate <= start);
  };

  return (
    <div className="box-border content-stretch flex flex-col gap-4 items-start justify-start p-0 relative shrink-0 w-[392px]">
      <div className="box-border content-stretch flex flex-row gap-1 h-6 items-center justify-between p-0 relative shrink-0 w-full">
        {/* Navigation Buttons - Left side */}
        {!hideLeftNav && (
        <div className="flex flex-row gap-1 items-center">
          {/* Prev Year */}
          <div 
            className="box-border content-stretch flex flex-row items-center justify-start p-0 relative rounded-[32px] shrink-0 cursor-pointer"
            onClick={() => onNavigateMonth('prev-year')}
          >
            <div className="overflow-clip relative shrink-0 size-6 rounded-[32px]">
              <div className="absolute bottom-1/4 left-[20.833%] right-[20.833%] top-1/4">
                <svg className="block size-full" fill="none" preserveAspectRatio="none" role="presentation" viewBox="0 0 14 12">
                  <g id="Vector">
                    <path d={svgPaths.p8039dc0} fill="#585858" />
                    <path d={svgPaths.p67fcb00} fill="#585858" />
                  </g>
                </svg>
              </div>
            </div>
          </div>
          {/* Prev Month */}
          <div 
            className="box-border content-stretch flex flex-row items-center justify-start p-0 relative rounded-[32px] shrink-0 cursor-pointer"
            onClick={() => onNavigateMonth('prev')}
          >
            <DatePickerMonthNavButton />
          </div>
        </div>
        )}
        
        {/* Month title is always shown */}
        <div className="basis-0 flex flex-col font-lato font-bold grow justify-center leading-[0] min-h-px min-w-px not-italic relative shrink-0 text-[#1c1c1c] text-[16px] text-center">
          <p className="block leading-[normal]">{monthTitle}</p>
        </div>
        
        {/* Navigation Buttons - Right side */}
        {!hideRightNav && (
        <div className="flex flex-row gap-1 items-center">
          {/* Next Month */}
          <div 
            className="box-border content-stretch flex flex-row items-center justify-start p-0 relative rounded-[32px] shrink-0 cursor-pointer"
            onClick={() => onNavigateMonth('next')}
          >
            <div className="overflow-clip relative shrink-0 size-6 rounded-[32px]">
              <div className="absolute bottom-1/4 left-[34.563%] right-[34.563%] top-1/4">
                <svg className="block size-full" fill="none" preserveAspectRatio="none" role="presentation" viewBox="0 0 8 12">
                  <g id="Vector">
                    <path d={svgPaths.p25284240} fill="#585858" />
                  </g>
                </svg>
              </div>
            </div>
          </div>
          {/* Next Year */}
          <div 
            className="box-border content-stretch flex flex-row items-center justify-start p-0 relative rounded-[32px] shrink-0 cursor-pointer"
            onClick={() => onNavigateMonth('next-year')}
          >
            <div className="overflow-clip relative shrink-0 size-6 rounded-[32px]">
              <div className="absolute bottom-1/4 left-[20.833%] right-[20.833%] top-1/4">
                <svg className="block size-full" fill="none" preserveAspectRatio="none" role="presentation" viewBox="0 0 14 12">
                  <g id="Vector">
                    <path d={svgPaths.p25284240} fill="#585858" />
                    <path d={svgPaths.p2d18c680} fill="#585858" />
                  </g>
                </svg>
              </div>
            </div>
          </div>
        </div>
        )}
      </div>
      
      <div className="box-border content-stretch flex flex-col items-center justify-start p-0 relative shrink-0 w-full">
        <DatePickerWeekLegend />
        
        {calendarData.map((week, weekIndex) => (
          <div key={weekIndex} className="box-border content-stretch flex flex-row items-center justify-center p-0 relative shrink-0">
            {week.map((day, dayIndex) => {
              if (day === null) {
                return <EmptyCell key={dayIndex} />;
              }
              
              const currentMonthYear = `${displayMonth}-${displayYear}`;
              const isStartDate = startDate && 
                startDate.day === day && 
                startDate.month === displayMonth && 
                startDate.year === displayYear;
                
              const isEndDate = endDate && 
                endDate.day === day && 
                endDate.month === displayMonth && 
                endDate.year === displayYear;
                
              const inRange = isDayInRange(day, currentMonthYear) || 
                isDayInHoverRange(day, currentMonthYear);
                
              const inRangeButNotEndpoint = inRange && !isStartDate && !isEndDate;
              
              const isHovered = hoverDate && 
                hoverDate.day === day && 
                hoverDate.month === displayMonth && 
                hoverDate.year === displayYear;
                
              return (
                <DatePickerDay 
                  key={dayIndex}
                  day={day}
                  month={currentMonthYear}
                  percentage={PERCENTAGES[day - 1]}
                  onClick={() => onDayClick(day, displayMonth, displayYear)}
                  isSelected={isStartDate || isEndDate}
                  isRangeStart={isStartDate && endDate !== null && (
                    startDate.day !== endDate.day || 
                    startDate.month !== endDate.month || 
                    startDate.year !== endDate.year
                  )}
                  isRangeEnd={isEndDate && startDate !== null && (
                    startDate.day !== endDate.day || 
                    startDate.month !== endDate.month || 
                    startDate.year !== endDate.year
                  )}
                  isInRange={inRangeButNotEndpoint}
                  onHover={() => onDayHover(day, displayMonth, displayYear)}
                  hoverDate={hoverDate}
                  startDate={startDate}
                  endDate={endDate}
                  isHovered={isHovered}
                  filterSettings={filterSettings}
                  selectedMetric={selectedMetric}
                />
              );
            })}
          </div>
        ))}
      </div>
    </div>
  );
}

// Main component
export default function DateRangePicker({ 
  allowMultipleMonths = false, // Metrics Calendar shows one month by default
  enableRangeSelection = true,
  selectionColor = "#E8EFFF"
}) {
  // Get current date to initialize with
  const today = new Date();
  const currentMonth = today.getMonth(); // 0-11
  const currentYear = today.getFullYear();
  
  // State for month navigation
  const [displayMonths, setDisplayMonths] = useState([
    { month: currentMonth, year: currentYear },
    { month: (currentMonth + 1) % 12, year: currentMonth === 11 ? currentYear + 1 : currentYear }
  ]);
  
  // State for date selection
  const [startDate, setStartDate] = useState(null);
  const [endDate, setEndDate] = useState(null);
  const [hoverDate, setHoverDate] = useState(null);
  
  // State for indicator filters
  const [showEvents, setShowEvents] = useState(true);
  const [showNotes, setShowNotes] = useState(true);
  const [showRestrictions, setShowRestrictions] = useState(true);
  const [showAnomalies, setShowAnomalies] = useState(true);
  
  const filterSettings = {
    showEvents,
    showNotes,
    showRestrictions,
    showAnomalies,
  };

  // state for metric selector
  const [selectedMetric, setSelectedMetric] = useState(METRIC_OPTIONS[0]);
  
  // Handle month navigation - always modifies both months together
  const handleMonthNavigation = (direction) => {
    setDisplayMonths(prevMonths => {
      // First month in the pair
      let { month, year } = prevMonths[0];
      
      switch(direction) {
        case 'prev':
          month = month - 1;
          if (month < 0) {
            month = 11;
            year -= 1;
          }
          break;
        case 'next':
          month = month + 1;
          if (month > 11) {
            month = 0;
            year += 1;
          }
          break;
        case 'prev-year':
          year -= 1;
          break;
        case 'next-year':
          year += 1;
          break;
      }
      
      // Calculate second month which is always one month ahead
      let nextMonth = month + 1;
      let nextYear = year;
      if (nextMonth > 11) {
        nextMonth = 0;
        nextYear += 1;
      }
      
      return [
        { month, year },
        { month: nextMonth, year: nextYear }
      ];
    });
  };
  
  // Handle day click for selection
  const handleDayClick = (day, month, year) => {
    if (!enableRangeSelection) {
      // Single date selection mode
      setStartDate({ day, month, year });
      setEndDate({ day, month, year });
      return;
    }
    
    if (!startDate || (startDate && endDate)) {
      // Start new selection
      setStartDate({ day, month, year });
      setEndDate(null);
    } else {
      // Complete the selection
      const newEndDate = { day, month, year };
      
      // Ensure start date is before end date
      const startDateTime = new Date(startDate.year, startDate.month, startDate.day);
      const newEndDateTime = new Date(year, month, day);
      
      if (newEndDateTime < startDateTime) {
        setEndDate(startDate);
        setStartDate(newEndDate);
      } else {
        setEndDate(newEndDate);
      }
    }
  };
  
  // Handle day hover for range preview
  const handleDayHover = (day, month, year) => {
    if (startDate && !endDate) {
      setHoverDate({ day, month, year });
    }
  };
  
  // Handle cancel button click
  const handleCancel = () => {
    setStartDate(null);
    setEndDate(null);
    setHoverDate(null);
  };
  
  // Handle apply button click
  const handleApply = () => {
    // Here you would typically pass the selected date range to a parent component
    console.log("Applied date range:", { 
      startDate: startDate ? { 
        ...startDate, 
        month: MONTHS[startDate.month] 
      } : null,
      endDate: endDate ? { 
        ...endDate, 
        month: MONTHS[endDate.month] 
      } : null
    });
  };
  
  // Handle shortcut selection
  const handleShortcutSelection = (range) => {
    setStartDate(range.startDate);
    setEndDate(range.endDate);
    
    // Update the calendar view to show the month of the start date
    setDisplayMonths([
      { month: range.startDate.month, year: range.startDate.year },
      { 
        month: range.startDate.month + 1 > 11 ? 0 : range.startDate.month + 1,
        year: range.startDate.month + 1 > 11 ? range.startDate.year + 1 : range.startDate.year
      }
    ]);
  };

  return (
    <div 
      className="bg-[#ffffff] box-border inline-flex flex-col items-start justify-start p-0 relative rounded shadow-[0px_1px_1px_0px_rgba(0,0,0,0.14),0px_2px_1px_-1px_rgba(0,0,0,0.12),0px_1px_3px_0px_rgba(0,0,0,0.2)] w-fit"
      style={{ "--fill-0": selectionColor, fontFamily: "Lato, sans-serif" }}
    >
      <div className="box-border content-stretch flex flex-col gap-4 items-start justify-start p-[24px] relative shrink-0 w-full">
        <div className="box-border content-stretch flex flex-row items-start justify-start p-0 relative shrink-0">
          {/* Calendar containers */}
          <div className="flex flex-col">
            <div className="box-border content-stretch flex flex-row items-start justify-start gap-12 p-0 relative shrink-0">
              {/* Render first month */}
              <Month 
                displayMonth={displayMonths[0].month}
                displayYear={displayMonths[0].year}
                startDate={startDate}
                endDate={endDate}
                onDayClick={handleDayClick}
                hoverDate={hoverDate}
                onDayHover={handleDayHover}
                onNavigateMonth={handleMonthNavigation}
                filterSettings={filterSettings}
                selectedMetric={selectedMetric}
                hideRightNav
              />

              {/* Second month */}
              <Month 
                displayMonth={displayMonths[1].month}
                displayYear={displayMonths[1].year}
                startDate={startDate}
                endDate={endDate}
                onDayClick={handleDayClick}
                hoverDate={hoverDate}
                onDayHover={handleDayHover}
                onNavigateMonth={handleMonthNavigation}
                filterSettings={filterSettings}
                selectedMetric={selectedMetric}
                hideLeftNav
              />
            </div>
          
            {/* Indicator filters */}
            <IndicatorFilterBar 
              showEvents={showEvents}
              setShowEvents={setShowEvents}
              showNotes={showNotes}
              setShowNotes={setShowNotes}
              showRestrictions={showRestrictions}
              setShowRestrictions={setShowRestrictions}
              showAnomalies={showAnomalies}
              setShowAnomalies={setShowAnomalies}
            />

            {/* Metric selector */}
            <MetricSelector selected={selectedMetric} onChange={setSelectedMetric} />
            
            {/* Actions bar - Moved here to appear below the metric selector */}
            <ActionsBar 
              onApply={handleApply}
              onCancel={handleCancel}
            />
          </div>
æ
          {/* Date Shortcuts */}
          <DateShortcuts 
            onSelectRange={handleShortcutSelection} 
            currentStartDate={startDate}
            currentEndDate={endDate}
          />
        </div>
      </div>
    </div>
  );
}

defineProperties(DateRangePicker, {
  allowMultipleMonths: {
    label: "Allow multiple months",
    type: "boolean",
    defaultValue: false
  },
  enableRangeSelection: {
    label: "Enable range selection",
    type: "boolean",
    defaultValue: true
  },
  selectionColor: {
    label: "Selection color",
    type: "string",
    defaultValue: "#E8EFFF"
  }
});
